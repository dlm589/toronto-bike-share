<script>
    import { onMount } from "svelte";
    import maplibregl from "maplibre-gl";
    import map_styles from "../lib/map-styles.json";
    import stations from "../data/Stations.geo.json";
    import bikeshare from "../data/Stations.geo.json"
    import * as d3 from "d3";

    let sheet = "num_bikes_available";
    let day = "0521";
    let days = ["0521", "0522", "0523", "0524", "0525", "0526"]
    let selectedDay = ""
    let capacity
    let station

    let value;

    let time = "1200";
    let map;
    let header = [];
    let csv = [];
    let dataLoaded= false

    // Esri color ramps - Heatmap 1
    const colors = ["#85c1c8ff", "#90a1beff", "#9c8184ff", "#a761aaff", "#af4980ff", "#b83055ff", "#c0182aff", "#c80000ff", "#d33300ff", "#de6600ff", "#e99900ff", "#f4cc00ff", "#ffff00ff"];
    // this is for handling the imported data and joining with the GIS layer
    async function handleCsvData(csv, header, day) {
        // Assuming 'MUNID' is the common key and find the index of regional and municipal layer count
        console.log(header)
        for (let i = 1; i < 1729; i++){
            let date = header[i].split("_")[0]

            if (date == day){
                //const commonField = header.indexOf(`${day}_${time}`);
                const commonField = header.indexOf(header[i]);
                stations.features.forEach((feature) => {
                // Find matching record in CSV data
                const matchingRecord = csv.find(
                    (obj) => obj['Station ID'] === feature.properties['Station ID'],
                );
                // If a match is found, update GeoJSON properties with CSV data
                if (matchingRecord) {
                    //console.log(parseInt(matchingRecord[`${day}_${time}`]))
                    //console.log(matchingRecord[`${day}_${time}`]/matchingRecord["Capacity"]*100)
                    //feature.properties[header[commonField]] = Math.round(matchingRecord[`${day}_${time}`]/matchingRecord["Capacity"]*10000)/100;
                    feature.properties[header[commonField]] = +matchingRecord[header[i]];

                }
            })
            }

            
        } 
        
    }

    function dayDropDown(){

    }


    onMount(async () => {
        // Load CSV data
        csv = await d3.csv(`data/${sheet}.csv`);
        console.log(csv)

        // get the header of the csv
        header = Object.keys(csv[0])   
        dataLoaded = handleCsvData(csv, header, day)

        console.log(stations)

            map = new maplibregl.Map({
                container: "map",
                style: map_styles, //'https://basemaps.cartocdn.com/gl/positron-gl-style/style.json',
                center: [-79.4, 43.65], // starting position
                minZoom: 11,
                maxZoom: 19,
                scrollZoom: true,
                attributionControl: false,
            });
        //console.log(csv[0]["Station ID"])
        if (dataLoaded){
            
        
            console.log(stations)
            map.on("load", () => {
                //adding the station, the data is determined either the "origin" or "destination" data input.
                map.addSource("station", {
                    type: "geojson",
                    data: stations,
                });
                /*map.addLayer({id: "bike-capacity-percentage",
                    type: "circle",
                    source: "station",
                    paint: {
                        "circle-radius": 5,
                        "circle-color": [
                            "interpolate",
                            ["linear"],
                            ["get", `${day}_${time}`],
                            0, colors[0],
                            10, colors[3],
                            20, colors[4],
                            30, colors[5],
                            40, colors[6],
                            50, colors[7],
                            60, colors[8],
                            70, colors[9],
                            80, colors[10],
                            90, colors[11],
                            100, colors[12]
                        ],
                        "circle-opacity": 0.7,
                        //"circle-stroke-color": "white",
                        //"circle-stroke-width": 2,
                    },
                });*/
                map.addLayer({id: "bike-count",
                type: "circle",
                    source: "station",
                    paint: {
                        "circle-radius": 5,
                        "circle-color": [
                            "interpolate",
                            ["linear"],
                            ["get", `${day}_${time}`],
                            0, colors[0],
                            5, colors[3],
                            10, colors[4],
                            15, colors[5],
                            20, colors[6],
                            25, colors[7],
                            30, colors[8],
                            35, colors[9],
                            40, colors[10],
                            45, colors[11],
                            50, colors[12]
                        ],
                        "circle-opacity": 0.7
                    },
                });


                map.on("mouseenter", "bike-count", (e) => {
                    map.getCanvas().style.cursor = "pointer";
                    console.log(e.features[0].properties[`${day}_${time}`])
                    station = e.features[0].properties['Name']
                    capacity = e.features[0].properties['Capacity']
                    value = e.features[0].properties[`${day}_${time}`]
                });

                map.on("mouseleave", "bike-count", () => {
                    map.getCanvas().style.cursor = "";
                });
            });

        }
    });
</script>

<div id="map" class="map" />

<div class = "info-panel">
    <h1>{station}: {value}</h1>
    <h2>Capacity: {capacity}</h2>
    <div id="select-wrapper">
        <select bind:value={selectedDay} on:change={dayDropDown}>
            {#each days as day}
                <option>{day}</option>
            {/each}
        </select>
    </div>
</div>

<style>
    .map {
        height: 75vh;
        width: 100vw;
        top: 0;
        left: 0%;
        position: absolute;
        overflow: hidden;
    }
    .info-panel {
        height: 25vh;
        width: 100vw;
        top: 75vh;
        left: 0;
        background-color: yellow;
        position: absolute;

    }
    select {
    width: 25vw;
    height: 25px;
    font-family: TradeGothicBold;
    font-size: 16px;
    color: #6d247a;
    border-width: 0px;
    margin-right: 5px;
    padding-left: 10px;
    padding-top: 3px;
    padding-bottom: 3px;
    border-radius: 0;
    border: 1px;
  }
</style>
