<script>
    import * as d3 from 'd3';
    import { legendColor } from 'd3-svg-legend';
    
    export let data;
    export let fullData;
    export let loading;
    export let variable;
    
    let width = 700;
    let height = 650;
    let svg;
    let g;
    let tooltip;
    

    let proj;
    let path;

    $: if (!loading) {
        proj = d3.geoMercator()
            .fitSize([550, 550], { type: 'FeatureCollection', features: data });
        path = d3.geoPath().projection(proj);
    }

    $: scale = !loading
        ? d3.scaleSequential(d3.interpolateRdBu)
            .domain([d3.min(data, d => d.properties[variable]), d3.max(data, d => d.properties[variable])])
        : null;

    let legend;
    $: if (!loading && scale) {
        const colorLegend = legendColor()
            .cells(9)
            .scale(scale)
            .title(variable);
        
        d3.select(legend).call(colorLegend);
    }

    function initializeZoom() {
        const zoom = d3.zoom()
            .scaleExtent([1, 8]) 
            .on('zoom', (event) => {
                d3.select(g).attr('transform', event.transform); 
            });

        d3.select(svg).call(zoom);
    }

    $: if (!loading && svg) {
        initializeZoom(); 
    }

    function showTooltip(event, d) {
        const [x, y] = d3.pointer(event);
        d3.select(tooltip)
            .style("left", `${x + 15}px`)
            .style("top", `${y + 15}px`)
            .style("opacity", 1)
            .html(`<strong>${d.properties.Name}</strong>
                <br>Total Population: ${d.properties["Total Population"]}
                <br>Median Age: ${d.properties["Median Age"]}
                <br>Percent White: ${d.properties["Percent White"]}
                <br>Percent Black or African American: ${d.properties["Percent Black or African American"]}
                <br>Percent American Indian and Alaska Native: ${d.properties["Percent American Indian and Alaska Native"]}
                <br>Percent Asian: ${d.properties["Percent Asian"]}
                <br>Percent Native Hawaiian and Other Pacific Islander: ${d.properties["Percent Native Hawaiian and Other Pacific Islander"]}
                <br>Percent Hispanic or Latino: ${d.properties["Percent Hispanic or Latino"]}
                <br>Median Income: ${d.properties["Median Income"]}
                <br>Percent of Population over 25 with Bachelor's Degree or Higher: ${d.properties["Percent of Population over 25 with Bachelor's Degree or Higher"]}
                <br>Total Housing Units: ${d.properties["Total Housing Units"]}
                `);
    }

    function hideTooltip() {
        d3.select(tooltip).style("opacity", 0);
    }
</script>
    
<main>
    <div bind:this={tooltip} class="tooltip"></div>
    <svg bind:this={svg} {width} {height}>
        <g bind:this={g}>
            {#each data as d}
                <path 
                    style="fill: {scale(+d.properties[variable])}"
                    d={path(d)}
                    on:mouseenter={(event) => showTooltip(event, d)}
                    on:mousemove={(event) => showTooltip(event, d)}
                    on:mouseleave={hideTooltip} />
            {/each}
            {#each fullData as d}
                <path class="geooverlay" d={path(d)} />
            {/each}
        </g>

        <g transform="translate({width - 150}, 50)" bind:this={legend} />
    </svg>
</main>
    
<style>
        .geooverlay {
            fill: none;
            stroke: black;
            stroke-width: 0.5px;
            stroke-opacity: 0.3;
        }

        .tooltip {
        position: absolute;
        background-color: white;
        border: 1px solid #ccc;
        padding: 8px;
        border-radius: 4px;
        font-size: 12px;
        pointer-events: none;
        opacity: 0;
        transition: opacity 0.2s ease;
        box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    }
</style>
    