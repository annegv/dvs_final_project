<script>
    import * as d3 from "d3";

    let arcGenerator = d3.arc().innerRadius(0).outerRadius(50);

    /**
	 * @type {any[]}
	 */
     export let data = [];

    let sliceGenerator = d3.pie().value(d => d.value);
    let arcData;
    /**
	 * @type {any}
	 */
    let arcs;
    $: {
        arcData = sliceGenerator(data);
        arcs = arcData.map(d => arcGenerator(d));
    }

    let colors = d3.scaleOrdinal(d3.schemeTableau10);

    export let selectedIndex = -1;

    let hoveredIndex = -1;
    let cursor = {x: 0, y: 0};

</script>
<div class="container">
	<svg width="70%" height="70%" viewBox="-50 -50 100 100">
        {#each arcs as arc, index}
            <path d={arc} fill={ colors(index) }
                class:selected={selectedIndex === index}
                on:click={e => selectedIndex = selectedIndex === index ? -1 : index} 
                on:mouseenter={evt => {
					hoveredIndex = index;
                    console.log("data", data);
					cursor = {x: evt.x, y: evt.y};
				}}
				on:mouseleave={evt => hoveredIndex = -1} />

            <!-- show the value and name if there is enough room. -->
			{#if (arcData[index].endAngle - arcData[index].startAngle) > 0.5}
                <text
                    style="font-weight: bold; font-size: 5px"
                    transform="translate({arcGenerator.centroid(arcData[index])})"
                    text-anchor="middle"
                    >
                    {data[index].label}
                </text>
				<text
                    style="font-size: 5px; margin: 0"
					text-anchor="middle"
					transform="translate({[arcGenerator.centroid(arcData[index])[0], arcGenerator.centroid(arcData[index])[1] + 7]})"
					>
					{arcData[index].value.toLocaleString("en-US")}
				</text>
			{/if}
        {/each}
    </svg>
    <ul class="legend" hidden={hoveredIndex === -1} style="top: {cursor.y}px; left: {cursor.x}">
        {#each data as d, index}
            <li style="--color: { colors(index) }" class:selected={selectedIndex === index}>
                <span class="swatch"></span>
                {d.label} <em>({d.value})</em>
            </li>
        {/each}
    </ul>
</div>     


<style>
    div.container {
        display: flex;
        gap: 1em;
    }

    svg {
        max-width: 20em;
        margin-block: 2em;

        /* Do not clip shapes outside the viewBox */
        overflow: visible;
    }

    /* svg:has(path:hover) {
        path:not(:hover) {
            opacity: 50%;
        }
    } */

    .selected {
        --color: oklch(60% 45% 0) !important;

        &:is(path) {
            fill: var(--color);
        }
    }

    path {
        transition: 300ms;
    }

    li {
        display: flex;
        align-items: center;
        gap: 0.5em;
    }

    ul.legend {
        flex: 1;
        list-style-type: none;
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(9em, 1fr));
        border-style: solid;
        border-color: #c9c9c9;
        padding: 0.5em;
        z-index: 1;
        background-color: white;

        &[hidden]:not(:hover, :focus-within) {
			opacity: 0;
			visibility: hidden;
		}
    }

    span.swatch {
        display: inline-block;
        width: 1em;
        height: 1em;
        background-color: var(--color);
    }
</style>