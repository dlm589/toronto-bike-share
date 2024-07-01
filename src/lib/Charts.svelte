<script>
    import { scaleLinear } from "d3-scale";
    import { rollups, sum } from "d3-array";
    import { LogoControl } from "maplibre-gl";
    //import data from "/src/data/growth/data.json";
    //import "../assets/global-styles.css";

    export let index;
    export let yTicks;
    export let colour;
    export let maxHeight;
    export let type;
    export let data;
    export let time;
    export let capacity;

    var features = data.features[index];
    var xTicks = Object.keys(features.properties); // get the values for the selected station
    xTicks.splice(xTicks.length - 2, 2); //remove name and capacity from the list

    var hour = [
        "00",
        "01",
        "02",
        "03",
        "04",
        "05",
        "06",
        "07",
        "08",
        "09",
        "10",
        "11",
        "12",
        "13",
        "14",
        "15",
        "16",
        "17",
        "18",
        "19",
        "20",
        "21",
        "22",
        "23",
    ];
    var minutes = [];

    xTicks.forEach((e, i) => {
        /*if (i%2 ===0){
            minutes.push(e.substring(7,8))
        }*/
        minutes.push(e.substring(7, 9));
    });

    var valueList = Object.values(features.properties);
    valueList.splice(valueList.length - 2, 2);
    var values = valueList;
    /*
    var values = []
    valueList.forEach((e, i) =>{
        if (i%2 === 0){
            values.push(e)
        }
    })
    */
    let width = 100;
    let height = 60;

    $: height = Math.min(width / 2.42, maxHeight);

    const padding = { top: 20, right: 25, bottom: 35, left: 15 };

    var barWidth = 14;

    var svgWidth = xTicks.length * barWidth + xTicks.length * 6;


    $: xScale = scaleLinear()
        .domain([0, minutes.length])
        .range([padding.left, svgWidth - padding.left - padding.right]);

    $: yScale = scaleLinear()
        .domain([0, Math.max.apply(0, yTicks)])
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
                                13 +
                                barPadding -
                                innerWidth / 600}
                            y={height - 5}
                            text-anchor="end"
                        >
                            {hr}
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

            <!--
        {#if type === "bar"}-->
            <g>
                {#each values as value, i}
                    <rect
                        class="barLight"
                        x={xScale(i) + barPadding}
                        y={yScale(1)}
                        width={barWidth - 2}
                        height={yScale(0) - yScale(1)}
                        stroke={colour}
                        opacity="0.15"
                        on:mouseover={(event) => {
                            selected_datapoint = value;
                            setMousePosition(event);
                            selected_datapoint_i = i;
                        }}
                        on:mouseout={() => {
                            selected_datapoint = undefined;
                        }}
                    />
                    <rect
                        class="bar"
                        x={xScale(i) + barPadding}
                        y={yScale(value / capacity)}
                        width={barWidth - 2}
                        height={yScale(0) - yScale(value / capacity)}
                        on:mouseover={(event) => {
                            selected_datapoint = value;
                            selected_datapoint_i = i;
                            // setMousePosition(event);
                        }}
                        on:mouseout={() => {
                            selected_datapoint = undefined;
                        }}
                        color={colour}
                    />
                {/each}
            </g>
            <!--{/if}
        -->
            <!-- y axis -->
            <g class="axis y-axis">
                {#each yTicks as tick}
                    <g
                        class="tick tick-{tick}"
                    >
                        <line x2="100%" />
                        <text y={yScale(tick)}>{tick * 100} </text>
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
            <!--
        {#if type === "line"}
            <g>
                {#each data as bike, i}
                    {console.log(bike)}
                    {#if i > 0 && values !== 0 && data[i - 1][variable] !== 0}
                        <line
                            x1={xScale(i - 1) + barPadding + barWidth/2 - 1}
                            y1={yScale(data[i - 1][variable])}
                            x2={xScale(i) + barPadding + barWidth/2 - 1}
                            y2={yScale(values)}
                            stroke={colour}
                            stroke-width="2"
                        />
                    {/if}
                    
                    {#if values !== 0}
                    {console.log(values)}
                        <circle
                            class="point"
                            r={innerWidth / 300}
                            cx={xScale(i) + barPadding + barWidth/2 - 1}
                            cy={yScale(values)}
                            fill={colour}
                            on:mouseover={(event) => {
                                selected_datapoint = bike;
                                selected_datapoint_i = i;
                                setMousePosition(event);
                            }}
                            on:mouseout={() => {
                                selected_datapoint = undefined;
                            }}
                        />

                        <rect
                            class="barLight"
                            x={xScale(i) + barPadding}
                            y={yScale(values)}
                            width={barWidth - 2}
                            height={yScale(0) - yScale(values)}
                            stroke={colour}
                            opacity=0.15
                            on:mouseover={(event) => {
                                selected_datapoint = bike;
                                setMousePosition(event);
                                selected_datapoint_i = i;
                            }}
                            on:mouseout={() => {
                                selected_datapoint = undefined;
                            }}
                        />
                    {/if}
                {/each}
            </g>
        {/if}-->
        </svg>
    </div>
</div>

<div id="hoverLabel">
    <p>
        {#if selected_datapoint != undefined}
        {Math.floor(selected_datapoint_i/12)}:{minutes[selected_datapoint_i]} - 
            <span id="lightBlue">{values[selected_datapoint_i]} bikes ({Math.round(values[selected_datapoint_i]/capacity*100)}% Capacity)</span>
        {/if}
    </p>
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
    }
    #hoverLabel {
        height: 30px;
        display: flex;
        align-items: center;
        justify-content: center;
    }
    #hoverLabel p {
        color: #F9F6F1;
        font-family: RobotoRegular;
        font-size: 16px;
        text-align: center;
    }
    #lightBlue {
        color: var(--brandLightBlue);
    }

    svg {
        position: relative;
    }

    .tick {
        font-family: RobotoRegular;
        font-size: 0.725em;
        font-weight: 200;
    }

    .tick line {
        stroke: #F9F6F1;
        stroke-width: 1px;
        opacity: 0.1;
    }
    .tick text {
        fill: #F9F6F1;
        text-anchor: start;
        font-size: 12px;
    }

    .tick.tick-0 line {
        stroke-dasharray: 0;
    }

    .x-axis .tick text {
        text-anchor: middle;
        font-size: 12px;
        text-align: right;
    }

    .x-label {
        text-anchor: middle;
        transform: translate(-10px, 0px) rotate(-90deg);
    }
    .x-label.tick text {
        font-size: 20px; /* Adjust the font size as desired */
        fill: #000000; /* Adjust the font color as desired */
    }

    .bar {
        stroke: #F9F6F1;
        stroke-width: 1px;
        stroke-opacity: 1;
        fill: #F9F6F1;
        cursor: pointer;
    }
    .bar:hover {
        fill: #e7adac;
        stroke: #e7adac;
        opacity: 1;
    }
    .barLight {
        stroke-width: 1px;
        stroke-opacity: 0;
    }

    .barLight:hover {
        stroke-width: 3px;
        stroke-opacity: 1;
        stroke: yellow;
    }

    .point {
        cursor: pointer;
    }
    .point:hover {
        fill: var(--brandLightBlue);
        opacity: 1;
    }
    .tip {
        stroke: var(--brandDarkGreen);
        stroke-width: 1px;
        fill: var(--brandYellow);
    }
    .year-tick {
        stroke-width: 2px;
        z-index: 6;
    }
</style>
