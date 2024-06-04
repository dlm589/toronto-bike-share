<script>
    import { onMount } from "svelte";
    import maplibregl from "maplibre-gl";
    import map_styles from "../lib/map-styles.json";
    import stations from "../data/Stations.geo.json";
    import * as d3 from "d3";

    let sheet = "num_bikes_available";
    let day = "0521";
    let time = "0900";
    let map;
    let header = [];
    let csv = [];
    let dataLoaded= false

    // Esri color ramps - Heatmap 1
    const colors = ["#85c1c8ff", "#90a1beff", "#9c8184ff", "#a761aaff", "#af4980ff", "#b83055ff", "#c0182aff", "#c80000ff", "#d33300ff", "#de6600ff", "#e99900ff", "#f4cc00ff", "#ffff00ff"];
    console.log(colors.length)
    // this is for handling the imported data and joining with the GIS layer
    async function handleCsvData(csv, header) {
        // Assuming 'MUNID' is the common key and find the index of regional and municipal layer count
        const commonField = header.indexOf(`${day}_${time}`);
        //console.log(commonField)
        try {
            stations.features.forEach((feature) => {
            // Find matching record in CSV data
            const matchingRecord = csv.find(
            (obj) => obj['Station ID'] === feature.properties['Station ID'],
            );
            // If a match is found, update GeoJSON properties with CSV data
            if (matchingRecord) {
                //console.log(parseInt(matchingRecord[`${day}_${time}`]))
                //console.log(matchingRecord[`${day}_${time}`]/matchingRecord["Capacity"]*100)
                feature.properties[header[commonField]] = Math.round(matchingRecord[`${day}_${time}`]/matchingRecord["Capacity"]*10000)/100;
            }
        });

        return true;
        // Now municipalities GeoJSON features have additional properties from the CSV data
        } catch (error) {
            console.error("Error fetching or processing CSV:", error);
        return false;
        }
    }




    onMount(async () => {
        // Load CSV data
        csv = await d3.csv(`data/${sheet}.csv`);
        console.log(csv)

        // get the header of the csv
        header = Object.keys(csv[0])   
        dataLoaded = handleCsvData(csv, header)

        console.log(stations)
        //console.log(csv[0]["Station ID"])
        if (dataLoaded){
            map = new maplibregl.Map({
                container: "map",
                style: map_styles, //'https://basemaps.cartocdn.com/gl/positron-gl-style/style.json',
                center: [-79.4, 43.65], // starting position
                minZoom: 11,
                maxZoom: 19,
                scrollZoom: true,
                attributionControl: false,
            });
        
            
            map.on("load", () => {
                //adding the station, the data is determined either the "origin" or "destination" data input.
                map.addSource("station", {
                    type: "geojson",
                    data: stations,
                });
                map.addLayer({
                    id: "station-layer",
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
                });
            });

        }
    });
</script>

<div id="map" class="map" />

<style>
    .map {
        height: 100vh;
        width: 100vw;
        top: 0;
        left: 0%;
        position: absolute;
        overflow: hidden;
    }
</style>
