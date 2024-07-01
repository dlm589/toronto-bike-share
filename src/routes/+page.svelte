<script>
    import { onMount } from "svelte";
    import maplibregl from "maplibre-gl";
    import map_styles from "../lib/map-styles.json";
    import Range from "../lib/Range.svelte";
    import Chart from "../lib/Charts.svelte";
    //import stations from "../data/Stations.geo.json";
    //import bikeshare from "../data/Stations.geo.json"
    import * as d3 from "d3";
    import bikes0521 from "../data/0521_num_bikes_available.geo.json"
    import bikes0522 from "../data/0522_num_bikes_available.geo.json"
    import bikes0523 from "../data/0523_num_bikes_available.geo.json"
    import bikes0524 from "../data/0524_num_bikes_available.geo.json"
    import bikes0525 from "../data/0525_num_bikes_available.geo.json"
    import bikes0526 from "../data/0526_num_bikes_available.geo.json"
    

    let days = ["0521", "0522", "0523", "0524", "0525", "0526"]
    let selectedDay = "0521"
    let station = "Union Station";
    let capacity = 43

    let daytime = "0521_0000"
    let bikecount;
    
    let map;
    let values = 0;
	let theme = "default";
    let stationNames = []
    let stationIndex = 31
    let hourMinutes = "00 : 00"
    //const colors = ["#85c1c8ff", "#90a1beff", "#9c8184ff", "#a761aaff", "#af4980ff", "#b83055ff", "#c0182aff", "#c80000ff", "#d33300ff", "#de6600ff", "#e99900ff", "#f4cc00ff", "#ffff00ff"];

    // #67001fff,#b2182bff,#d6604dff,#f4a582ff,#fddbc7ff,#f7f7f7ff,#d1e5f0ff,#92c5deff,#4393c3ff,#2166acff,#053061ff
    const colors = ["#67001fff", "#b2182bff", "#d6604dff", "#f4a582ff", "#fddbc7ff", "#f7f7f7ff", "#d1e5f0ff", "#92c5deff", "#4393c3ff", "#2166acff", "#053061ff"];
    const bikeshare = {
        "0521" : bikes0521,
        "0522" : bikes0522,
        "0523" : bikes0523,
        "0524" : bikes0524,
        "0525" : bikes0525,
        "0526" : bikes0526
    }
    let circlecolor_perc = [
        "interpolate",
        ["linear"],
        //["get", daytime], 
        ['/', ['get', daytime], ['get', 'Capacity']],
        0, colors[0], 
        0.10, colors[1], 0.20, colors[2], 0.30, colors[3], 
        0.40, colors[4], 0.50, colors[5], 0.60, colors[6], 0.70, colors[7], 
        0.80, colors[8], 0.90, colors[9], 
        1.00, colors[10]]
    let circlecolor_count = [
        "interpolate",
        ["linear"],
        ["get", daytime],
        0, colors[0], 5, colors[1], 10, colors[2], 15, colors[3], 20, colors[4],
        25, colors[5], 30, colors[6], 35, colors[7], 40, colors[8], 45, colors[9],
        50, colors[10]]

    async function dayDropDown(){
        //console.log(selectedDay)
        
        map.getSource("station").setData(bikeshare[selectedDay]);
        
        var hr = daytime.split("_")[1]
        daytime = `${selectedDay}_${hr}`
        //console.log(hr)
        //console.log(daytime)
    }

    function valueTime(value, day, station){

        const hours = Math.floor(value*5 / 60);
        const minutes = value*5 % 60;
        if (minutes <10 && hours <10){
            hourMinutes = `0${hours} : 0${minutes}`
            daytime = `${day}_0${hours}0${minutes}`
            circlecolor_perc[2] = ['/', ['get', daytime], ['get', 'Capacity']]
        } else if (minutes < 10){
            hourMinutes = `${hours} : 0${minutes}`
            daytime = `${day}_${hours}0${minutes}`
            circlecolor_perc[2] = ['/', ['get', daytime], ['get', 'Capacity']]
        } else if (hours < 10){
            hourMinutes = `0${hours} : ${minutes}`
            daytime = `${day}_0${hours}${minutes}`
            circlecolor_perc[2] = ['/', ['get', daytime], ['get', 'Capacity']]
        } else {
            hourMinutes = `${hours} : ${minutes}`
            daytime = `${day}_${hours}${minutes}`
            
            circlecolor_perc[2] = ['/', ['get', daytime], ['get', 'Capacity']]
        }
        stationIndex = stationNames.indexOf(station)
        bikecount = bikeshare[selectedDay].features[stationIndex].properties[daytime]

        map.setPaintProperty("bike-count", 'circle-radius', 5);
        map.setPaintProperty("bike-count", 'circle-color', circlecolor_perc);
        map.setPaintProperty("bike-count", 'circle-opacity', 0.7);

        //console.log(daytime)

        return hourMinutes
    }

    onMount(async () => {
        // Load CSV data
        ///csv = await d3.csv(`data/${sheet}.csv`);
        //console.log(csv)

        // get the header of the csv
        //header = Object.keys(csv[0])   
        //dataLoaded = handleCsvData(csv, header, day)

        //console.log(csv[0]["Station ID"])
        console.log(bikes0521)
            
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
                
                map.addLayer({id: "bike-count",
                type: "circle",
                    source: "station",
                    paint: {
                        "circle-radius": 5,
                        "circle-color": circlecolor_perc,
                        "circle-opacity": 0.7
                    },
                });

                map.on("click", "bike-count", (e) => {
                    station = e.features[0].properties['Name']
                    console.log(station)
                    capacity = e.features[0].properties['Capacity']
                    bikecount = e.features[0].properties[daytime]
                    stationIndex = stationNames.indexOf(station)
                });
                map.on("mouseenter", "bike-count", (e) => {
                    map.getCanvas().style.cursor = "pointer";
                });
                map.on("mouseleave", "bike-count", () => {
                    map.getCanvas().style.cursor = "";
                });

                for (let i = 0; i < bikes0521.features.length; i++){
                    //console.log(i,bikes0521.features[i].properties.Name)
                    stationNames.push(bikes0521.features[i].properties.Name)
                }

                stationIndex = stationNames.indexOf(station)
                bikecount = bikes0521.features[stationIndex].properties[daytime]
            });
    });
</script>

<div id="map" class="map" />

<div class = "info-panel">
    <h1>{station} at {hourMinutes}: </h1>
    <h2>{bikecount} / {capacity} ({Math.round(bikecount/capacity*10000)/100}%) </h2>

    <!-- Slider -->
    <div class:purple-theme={theme === "default"}>
        <Range on:change={(e) => {
            values = e.detail.value
            console.log(values)
            valueTime(values, selectedDay, station)
            }} 
            id="basic-slider" />
    </div>
    
    <div id="select-wrapper">
        <select bind:value={selectedDay} on:change={dayDropDown}>
            {#each days as day}
                <option>{day}</option>
            {/each}
        </select>
    </div>
    
    {#key stationIndex, bikeshare[selectedDay]}
    <Chart on:change = {(e) => console.log(e)}
        index= {stationIndex}
        data = {bikeshare[selectedDay]}
        yTicks={[0, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0]}
        colour="#F1C500"
        maxHeight="250"
        type="line"
        time = {daytime}
        capacity = {capacity}
    />
    {/key}
    
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
    .info-panel {
        height: 45vh;
        width: 100vw;
        top: 55vh;
        left: 0;
        background-color: #373737;
        position: absolute;
        overflow-x: hidden;

        

    }
    select {
        width: 25vw;
        height: 25px;
        font-family: TradeGothicBold;
        font-size: 16px;
        color: #6d247a;
        border-width: 0px;
        margin-right: 5px;
        margin-left: 10px;
        padding-left: 10px;
        padding-top: 3px;
        padding-bottom: 3px;
        border-radius: 0;
        border: 1px;
  }
  h1 {
    margin-left: 10px;
    color: #F9F6F1;
  }
  h2{
    margin-left: 10px;
    color: #F9F6F1;
  }
</style>
