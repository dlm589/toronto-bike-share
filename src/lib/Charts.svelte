<script>
    import { scaleLinear } from "d3-scale";
    import { createEventDispatcher } from "svelte";
    import { rollups, sum } from "d3-array";
    import { LogoControl } from "maplibre-gl";

    export let index;
    export let yTicks;
    export let maxHeight;
    export let data;
    export let capacity;

    let value = 30
    let selecttime = "00:00"
    let selected_i = 0
    // Dispatch 'change' events
    const dispatch = createEventDispatcher();

    // Allows both bind:value and on:change for parent value retrieval
    function setValue(val, selectedPoint_i) {
      value = val;
      if (selectedPoint_i%12 < 2){
        selecttime = `${Math.floor(selectedPoint_i/12)}`+" : 0"+`${selectedPoint_i%12*5}`
      } else {
        selecttime = `${Math.floor(selectedPoint_i/12)}`+" : "+`${selectedPoint_i%12*5}`
      }
        selected_i = selectedPoint_i
      dispatch("change", { value, selecttime , selected_i});
    }


    var features = data.features[index];
    var xTicks = Object.keys(features.properties); // get the values for the selected station
    xTicks.splice(xTicks.length - 2, 2); //remove name and capacity from the list

    var hour = [
        "00","01","02","03",
        "04","05","06","07",
        "08","09","10","11",
        "12","13","14","15",
        "16","17","18","19",
        "20","21","22","23",
    ];
    var minutes = [];

    xTicks.forEach((e, i) => {
        minutes.push(e.substring(7, 9));
    });

    var valueList = Object.values(features.properties);
    valueList.splice(valueList.length - 2, 2);
    var values = valueList;

    let width = 100;
    let height = 600;

    $: height = Math.min(width / 2.42, maxHeight);

    const padding = { top: 20, right: 25, bottom: 35, left: 15 };

    var barWidth = 14;

    // this is for calculating the width of the chart 
    // calculating the sum of the bar widths and the gaps between the bars
    var svgWidth = xTicks.length * barWidth + xTicks.length * 6;


    $: xScale = scaleLinear()
        .domain([0, minutes.length])
        .range([padding.left, svgWidth - padding.left - padding.right]);

    $: yScale = scaleLinear()
        .domain([0, Math.max(...yTicks)])
        .range([height - padding.bottom, padding.top]);

    $: innerWidth = width - (padding.left + padding.right);

    //$: barWidth = Math.max(Math.min(innerWidth / xTicks.length, 9), 14);

    var barPadding = 10; // controls how much spacing the bars will be from the

    let selected_datapoint = undefined;
    let selected_datapoint_i = undefined;

    let mouse_x, mouse_y;
    const setMousePosition = function (event) {
        mouse_x = event.clientX;
        mouse_y = event.clientY;
    };

</script>

<div class="chart-container">
    <div id="barchart" class="chart" bind:clientWidth={width}>
        <svg width={svgWidth} {height}>
            <g class="year-tick">
                {#each hour as hr, i}
                    <line
                        class="year-grid"
                        x1={xScale(i * 12) + barPadding-3}
                        y1={height - 3}
                        x2={xScale(i * 12) + barPadding-3}
                        y2={0}
                        stroke-width={3}
                        stroke="#EBD9CB"
                        stroke-dasharray="5 3"
                        opacity="0.5"
                    />
                {/each}
            </g>
            <g class="axis x-axis">
                {#each hour as hr, i}
                    <g class="tick">
                        <text
                            x={xScale(i * 12) +
                                18 +
                                barPadding -
                                innerWidth / 600}
                            y={height - 5}
                            text-anchor="end"
                        >
                            {hr} hr
                        </text>
                    </g>
                {/each}
            </g>
            <!--This is the X Axis-->

            <g class="axis x-axis">
                {#each minutes as mn, i}
                    <g class="tick" transform="translate({xScale(i)},{height})">
                        <text x={barWidth / 2 + 11} y="-20">{mn}</text>
                    </g>
                {/each}
            </g>
            <g>
                {#each values as value, i}
                    
                    <rect
                        class="bar"
                        x={xScale(i) + barPadding}
                        y={yScale(value / capacity)}
                        width={barWidth}
                        height={Math.max(yScale(0) - yScale(value / capacity), 0)}
                        on:mouseover={(event) => {
                            selected_datapoint = value;
                            selected_datapoint_i = i;
                            setValue(selected_datapoint, selected_datapoint_i)
                        }}
                        on:mouseout={() => {
                            selected_datapoint = undefined;
                        }}

                    />

                    <rect
                        class="barLight"
                        x={xScale(i) + barPadding}
                        y={yScale(1)}
                        width={barWidth}
                        height={Math.max(yScale(0) - yScale(1),0)}

                        on:mouseover={(event) => {
                            selected_datapoint = value;
                            setMousePosition(event);
                            selected_datapoint_i = i;
                            setValue(selected_datapoint, selected_datapoint_i)
                        }}
                        on:mouseout={() => {
                            selected_datapoint = undefined;
                        }}

                        
                    />
                   
                {/each}
            </g>
            <!-- y axis -->
            <g class="axis y-axis">
                {#each yTicks as tick}
                    <g
                        class="tick" 
                    >
                        <text y={yScale(tick)+3} >{tick * 100} </text>
                    </g>
                    <line
                        class="year-grid"
                        y1={yScale(tick)}
                        x1={15 + barPadding}
                        y2={yScale(tick)}
                        x2={svgWidth}
                        stroke-width={1}
                        stroke="black"
                        stroke-dasharray="5 3"
                        opacity="0.5"
                    />
                {/each}
            </g>
        </svg>
    </div>
</div>

<style>
    .chart-container {
        overflow-x: auto;
    }
    
    .chart {
        width: 100%;
        min-width: 300px;
        margin: 0 auto;
        margin-left: 10px;
        margin-right: 10px;
        text-align: right;
    }

    svg {
        position: relative;
        text-align: right;
    }

    .tick {
        font-family: RobotoRegular;
        font-size: 0.725em;
        font-weight: 200;
        text-align: right;
    }

    .tick line {
        stroke: #F9F6F1;
        text-align: right;
        stroke-width: 1px;
        opacity: 1;
    }

    .tick text {
        fill: #F9F6F1;
        text-anchor: start;
        font-size: 12px;
        text-align: right;
    }

    .tick.tick-0 line {
        stroke-dasharray: 0;
        text-align: right;
    }

    .x-axis .tick text {
        text-anchor: middle;
        font-size: 12px;
        text-align: right;
    }

    .x-label {
        text-anchor: middle;
        transform: translate(-10px, 0px) rotate(-90deg);
        text-align: right;
    }

    .x-label.tick text {
        font-size: 20px; /* Adjust the font size as desired */
        fill: #000000; /* Adjust the font color as desired */
        text-align: right;
    }

    .bar {

        fill: white;
        opacity: 0.5;
        cursor: pointer;
    }

    .barLight {
        opacity: 0;
    }

    .barLight:hover {
        stroke: #FFBF00;
        fill: rgba(0,0,0,0);
        stroke-width: 2px;
        opacity: 1;
    }
    
    .year-tick {
        stroke-width: 2px;
        z-index: 6;
    }
</style>
