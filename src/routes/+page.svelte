<script>
    import * as d3 from "d3";
    import Pie from "$lib/Pie.svelte";
	import { onMount } from "svelte";
    // import income from '$lib/municipality_median_incomes.json';

    /**
	 * @type {string[]}
	 */
    let mbta_communities;
    let municipiality_income_data = {};
    let municipiality_race_data = {};
    let municipiality_commute_data = {};
    let selected_municipal = ""

    let isLoading = true;

    async function fetch_municipalities() {
        try {
            const response = await fetch("src/data/mbta_municipalities.txt");
            const text = await response.text();
            mbta_communities = text.split('\n').map(municipality => municipality.trim());
            isLoading = false;
        } catch (error) {
            console.error('Error fetching data:', error);
        }
    }

    async function fetch_income() {
        let income_data = await d3.json("src/data/municipality_income_quantiles.json")
        income_data.forEach(row => {
            municipiality_income_data[row.MUNICIPALITY.replace(/ town|city|[" ]/g, '').toLowerCase()] = [
                { value: Number(row.QUINTILE_1/100).toFixed(2), label: "Q1" },
                { value: Number(row.QUINTILE_2/100).toFixed(2), label: "Q2" },
                { value: Number(row.QUINTILE_3/100).toFixed(2), label: "Q3" },
                { value: Number(row.QUINTILE_4/100).toFixed(2), label: "Q4" },
                { value: Number(row.QUINTILE_5/100).toFixed(2), label: "Q5" },
            ];
        });
    }

    async function fetch_race(){
        let race_data = await d3.json("src/data/municipality_race_demographics.json")
        race_data.forEach(row => {
            municipiality_race_data[row.MUNICIPALITY.replace(/ town|city|[" ]/g, '').toLowerCase()] = [
                { value: Number(row.WHITE).toFixed(2), label: "White" },
                { value: Number(row.BLACK).toFixed(2), label: "Black" },
                { value: Number(row.NATIVE_AMERICAN).toFixed(2), label: "AI/AN" },
                { value: Number(row.ASIAN).toFixed(2), label: "Asian" },
                { value: Number(row.HAWAIIAN_PACIFIC_ISLANDER).toFixed(2), label: "NH/PI" },
                { value: Number(row.OTHER).toFixed(2), label: "Other" },
                { value: Number(row.TWO_OR_MORE_RACES).toFixed(2), label: "2+ Races" },
            ]
        })
    }

    async function fetch_commute(){
        let commute_data = await d3.json("src/data/municipality_work_transport_modes.json")
        commute_data.forEach(row => {
            municipiality_commute_data[row.MUNICIPALITY.replace(/ town|city|[" ]/g, '').toLowerCase()] = [
                { value: Number(row.VEHICLE_ALONE).toFixed(2), label: "Vehicle" },
                { value: Number(row.VEHICLE_CARPOOL).toFixed(2), label: "Carpool" },
                { value: Number(row.PUBLIC_BUS).toFixed(2), label: "Bus" },
                { value: Number(row.PUBLIC_SUBWAY).toFixed(2), label: "Subway" },
                { value: Number(row.PUBLIC_RAIL).toFixed(2), label: "Rail" },
                { value: Number(row.BICYCLE).toFixed(2), label: "Bike" },
                { value: Number(row.WALK).toFixed(2), label: "Walk" },
                { value: Number(row.OTHER).toFixed(2), label: "Other" },
            ]
        })
    }

    let handleSelectedMunicipal = (e) => {
        selected_municipal = e.target.value;
        // console.log(selected_municipal)
    };
    
    onMount(async () => {
        fetch_municipalities()
        fetch_income()
        fetch_race()
        fetch_commute()
    })
</script>

<h1 class='center-align'>Final Project Visualization</h1>
<h2 class='center-align'>Data by Municipality</h2>
<fieldset class='center-align'>
    {#if isLoading}
    <p>Loading...</p>
    {:else}
    <legend>Select a municipality:</legend> 
    <select id="municipalities" on:change={handleSelectedMunicipal}>
        <option value=""/>
        {#each Object.entries(mbta_communities) as [key, value]}
            <option value={value}>{value}</option>
        {/each}
    </select>
    {/if}
</fieldset>
{#if selected_municipal}
    {#key selected_municipal}
    <div class="pie-container">
        <div>
            <h3>Income</h3>
            <Pie data={municipiality_income_data[selected_municipal.toLowerCase()]}/>
        </div>
        <div>
            <h3>Race</h3>
            <Pie data={municipiality_race_data[selected_municipal.toLowerCase()]}/>
        </div>
        <div>
            <h3>Commute Type</h3>
            <Pie data={municipiality_commute_data[selected_municipal.toLowerCase()]}/>
        </div>
    </div>
    {/key}
{/if}

<style>
    .pie-container {
        display: grid;
        grid-template-columns: 30% 30% 30%;
        max-width: 100%;
        justify-content: center;
    }

    .center-align {
        justify-items: center;
        text-align: center;
        justify-content: center;
    }
</style>