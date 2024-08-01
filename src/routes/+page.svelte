<script>
    import { onMount } from "svelte";
    import maplibregl from "maplibre-gl";
    import map_styles from "../lib/map-styles.json";
    import Range from "../lib/Range.svelte";
    import Chart from "../lib/Charts.svelte";
    //import stations from "../data/Stations.geo.json";
    //import bikeshare from "../data/Stations.geo.json"
    import * as d3 from "d3";
    import "../assets/global-styles.css";
    import bikes0521 from "../data/0521_num_bikes_available.geo.json";
    import bikes0522 from "../data/0522_num_bikes_available.geo.json";
    import bikes0523 from "../data/0523_num_bikes_available.geo.json";
    import bikes0524 from "../data/0524_num_bikes_available.geo.json";
    import bikes0525 from "../data/0525_num_bikes_available.geo.json";
    import bikes0526 from "../data/0526_num_bikes_available.geo.json";

    let days = [
        "Tuesday, 05/21/2024",
        "Wednesday, 05/22/2024",
        "Thursday, 05/23/2024",
        "Friday, 05/24/2024",
        "Saturday, 05/25/2024",
        "Sunday, 05/26/2024",
    ];
    let selectedDay = "Tuesday, 05/21/2024";
    let station = "Union Station";
    let capacity = 43;

    let daytime = "0521_0000";
    let bikecount;

    let map;
    let values = 0;
    let theme = "default";
    let stationNames = [];
    let stationIndex = 31;
    let hourMinutes = "00 : 00";

    const colors = [
        "#f5fff2ff",
        "#b7e3b2ff",
        "#72c66dff",
        "#5b9756ff",
        "#43683fff",
        "#3d5439ff",
    ];
    const bikeshare = {
        "0521": bikes0521,
        "0522": bikes0522,
        "0523": bikes0523,
        "0524": bikes0524,
        "0525": bikes0525,
        "0526": bikes0526,
    };

    let circlecolor_perc = [
        "interpolate",
        ["linear"],
        //["get", daytime],
        ["/", ["get", daytime], ["get", "Capacity"]],
        0,
        colors[0], //0.10, colors[1],
        0.2,
        colors[1], //0.30, colors[3],
        0.4,
        colors[2], //0.50, colors[5],
        0.6,
        colors[3], //0.70, colors[7],
        0.8,
        colors[4], //0.90, colors[9],
        1.0,
        colors[5],
    ];
    let circlecolor_count = [
        "interpolate",
        ["linear"],
        ["get", daytime],
        0,
        colors[0],
        5,
        colors[1],
        10,
        colors[2],
        15,
        colors[3],
        20,
        colors[4],
        25,
        colors[5],
        30,
        colors[6],
        35,
        colors[7],
        40,
        colors[8],
        45,
        colors[9],
        50,
        colors[10],
    ];

    function convertSelectedDay(sDay) {
        sDay = sDay.split(", ")[1];
        sDay = sDay.replace("/2024", "");
        sDay = sDay.replace("/", "");

        return sDay;
    }

    async function dayDropDown() {
        // this function gets the selected day and updates the map layers

        //get the value of the selected day
        var sday = convertSelectedDay(selectedDay);
        var hr = daytime.split("_")[1];
        daytime = `${sday}_${hr}`;
        //update day in circlecolor_perc
        circlecolor_perc[2] = ["/", ["get", daytime], ["get", "Capacity"]];

        map.getSource("station").setData(bikeshare[sday]);

        map.setPaintProperty("bike-count", "circle-radius", 5);
        map.setPaintProperty("bike-count", "circle-color", circlecolor_perc);
        map.setPaintProperty("bike-count", "circle-opacity", 0.7);
    }

    function valueTime(value, day, station) {
        console.log(value);
        var sday = convertSelectedDay(selectedDay);
        const hours = Math.floor((value * 5) / 60);
        const minutes = (value * 5) % 60;
        if (minutes < 10 && hours < 10) {
            hourMinutes = `0${hours} : 0${minutes}`;
            daytime = `${day}_0${hours}0${minutes}`;
            circlecolor_perc[2] = ["/", ["get", daytime], ["get", "Capacity"]];
        } else if (minutes < 10) {
            hourMinutes = `${hours} : 0${minutes}`;
            daytime = `${day}_${hours}0${minutes}`;
            circlecolor_perc[2] = ["/", ["get", daytime], ["get", "Capacity"]];
        } else if (hours < 10) {
            hourMinutes = `0${hours} : ${minutes}`;
            daytime = `${day}_0${hours}${minutes}`;
            circlecolor_perc[2] = ["/", ["get", daytime], ["get", "Capacity"]];
        } else {
            hourMinutes = `${hours} : ${minutes}`;
            daytime = `${day}_${hours}${minutes}`;

            circlecolor_perc[2] = ["/", ["get", daytime], ["get", "Capacity"]];
        }
        stationIndex = stationNames.indexOf(station);
        bikecount = bikeshare[sday].features[stationIndex].properties[daytime];

        map.setPaintProperty("bike-count", "circle-radius", 5);
        map.setPaintProperty("bike-count", "circle-color", circlecolor_perc);
        map.setPaintProperty("bike-count", "circle-opacity", 0.7);

        return hourMinutes;
    }

    onMount(async () => {
        map = new maplibregl.Map({
            container: "map",
            style: map_styles, //'https://basemaps.cartocdn.com/gl/positron-gl-style/style.json',
            center: [-79.4, 43.65], // starting position
            minZoom: 11,
            maxZoom: 19,
            scrollZoom: true,
            attributionControl: false,
        });

        //console.log(stations)
        map.on("load", () => {
            //adding the station, the data is determined either the "origin" or "destination" data input.
            map.addSource("station", {
                type: "geojson",
                data: bikes0521,
            });

            map.addLayer({
                id: "bike-count",
                type: "circle",
                source: "station",
                paint: {
                    "circle-radius": 5,
                    "circle-stroke-width": 0.4,
                    "circle-stroke-color": "#0D534D",
                    "circle-color": circlecolor_perc,
                    "circle-opacity": 0.7,
                },
            });

            map.addLayer({
                id: "bike-clicked",
                type: "circle",
                source: "station",
                filter: ["==", "Name", station],
                paint: {
                    "circle-radius": 5,
                    "circle-opacity": 0,
                    "circle-stroke-width": 3,
                    "circle-stroke-color": "#DC4633",
                },
            });

            map.on("click", "bike-count", (e) => {
                station = e.features[0].properties["Name"];
                map.setFilter("bike-clicked", ["==", "Name", station]),
                    (capacity = e.features[0].properties["Capacity"]);
                bikecount = e.features[0].properties[daytime];
                stationIndex = stationNames.indexOf(station);
            });
            map.on("mouseenter", "bike-count", (e) => {
                map.getCanvas().style.cursor = "pointer";
            });
            map.on("mouseleave", "bike-count", () => {
                map.getCanvas().style.cursor = "";
            });

            for (let i = 0; i < bikes0521.features.length; i++) {
                //console.log(i,bikes0521.features[i].properties.Name)
                stationNames.push(bikes0521.features[i].properties.Name);
            }

            stationIndex = stationNames.indexOf(station);
            bikecount = bikes0521.features[stationIndex].properties[daytime];
        });
    });
</script>

<div id="maps">
    <div id="map" class="map" />
    <div class="legend">
        <div class="legend-gradient"></div>
        <div class="legend-labels">
            <span style="text-align:left;">0%</span>

            <span style="text-align:right;">100%</span>
        </div>
    </div>
</div>
<div class="info-panel">
    <h1>{station} Bike Availability</h1>

    <div class="header">
        <h2>On</h2>
        &nbsp
        <div id="select-wrapper">
            <select bind:value={selectedDay} on:change={dayDropDown}>
                {#each days as day}
                    <option>{day}</option>
                {/each}
            </select>
        </div>
        &nbsp
        <h2>
            at {hourMinutes} - {bikecount} bikes / {capacity} docks ({Math.round(
                (bikecount / capacity) * 10000,
            ) / 100}%)
        </h2>
    </div>
    {#key (stationIndex, bikeshare[convertSelectedDay(selectedDay)])}
        <Chart
            on:change={(e) => {
                bikecount = e.detail.value;
                hourMinutes = e.detail.selecttime;
                values = e.detail.selected_i;
                valueTime(values, convertSelectedDay(selectedDay), station);
            }}
            index={stationIndex}
            data={bikeshare[convertSelectedDay(selectedDay)]}
            yTicks={[0, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0]}
            maxHeight="250"
            {capacity}
        />
    {/key}

    <!-- Slider -->
    <h2>Time Slider {hourMinutes}</h2>
    <div class:purple-theme={theme === "default"}>
        <Range
            on:change={(e) => {
                values = e.detail.value;
                valueTime(values, convertSelectedDay(selectedDay), station);
            }}
            id="basic-slider"
        />
    </div>
    <h1>About This Project</h1>
    <p>
        Biking has become increasingly more convenient in the city. Bike Share Toronto has also seen its ridership increase every year. As a Bike Share user, I really enjoy the convenience of this system. However, there are times when it is difficult to find a bike to borrow or an empty dock to return a bike to. To understand this issue on a larger scale, I collected Bike Share Toronto's bike availability data at 5-minute intervals for (almost) a week in May and tried to visualize the data. The information is pulled from Bike Share Toronto's API.
    </p>
    <h2>What Did I See? Population Flow!</h2>
    <p>
        From playing around with the slider and scrolling through the bar chart, the changing colors of the bike stations show influxes of Toronto's population in and out of Downtown Toronto. On weekdays, the bikes are dispersed in the morning and gradually flow into Downtown Toronto during the morning rush hour. It can be clearly seen that after 9:00 AM, many of the Bike Share stations are at 100% capacity while bike stations in the inner residential suburbs are nearly empty. The stations remain at nearly full capacity, with minor changes during lunch hours (12-1) as users might ride bikes to move within Downtown. However, these users can have a more difficult time returning the bikes as most stations are still at full or nearly full capacity. The next major flow starts around 5:00 PM, when bikes are taken out of Downtown. By that time, Downtown's Bike Share stations are empty or almost empty and will continue to have low bike availability until the rush hour the next morning. Meanwhile, in the inner suburbs of Downtown Toronto, stations are gradually filled up.
    </p>
    <p></p>
</div>

<style>
    .map {
        height: 55vh;
        width: 100vw;
        top: 0;
        left: 0%;
        position: absolute;
        overflow: hidden;
    }

    .legend {
        position: absolute;
        top: 5px;
        right: 5px;
        background-color: rgba(255, 255, 255, 0.8);
        padding: 10px;
        border-radius: 5px;
        box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
        z-index: 10;
        text-align: center;
        width: 20%;
    }

    .legend-title {
        margin-bottom: 5px;
        font-size: 14px;
        font-weight: bold;
    }

    .legend-gradient {
        height: 10px;
        background: linear-gradient(
            to right,
            #f5fff2ff,
            #b7e3b2ff,
            #72c66dff,
            #5b9756ff,
            #43683fff,
            #3d5439ff
        );
        border-radius: 3px;
        margin-bottom: 5px;
    }

    .legend-labels {
        display: flex;
        justify-content: space-around;
        font-size: 12px;
    }

    .legend-labels span {
        flex: 1;
        text-align: center;
    }
    .info-panel {
        height: 45vh;
        width: 100vw;
        top: 55vh;
        left: 0;
        background-color: rgba(55, 55, 55);
        position: absolute;
        overflow-x: hidden;
    }
    .header {
        display: flex;
        align-items: center;
        padding-left: 10px;
        color: #f9f6f1;
        font-size: 20px;
    }

    select {
        width: auto;
        height: auto;
        font-size: 20px;
        color: var(--brandYellow);
        font-family: TradeGothicBold;
        background: rgba(55, 55, 55);
        border: rgba(55, 55, 55);
    }
    h2 {
        padding-left: 10px;
    }
    p {
        left: 10px;
        padding-left: 10px;
        padding-right: 10px;
    }
</style>
