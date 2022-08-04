<script>
    import Chart from "chart.js/auto";
    import { onMount } from "svelte";
    import Footer from "./components/Footer.svelte";
    import Header from "./components/Header.svelte";
    import orgData from "./data/data.json";

    // SORTING DATA
    let jsonData = orgData.map((doc) => ({
        ...doc,
        properties: doc.properties.map((prop) => ({
            ...prop,
            data: prop.data
                .map((row) => ({
                    ...row,
                    d: new Date(row.d),
                }))
                .sort((a, b) => a.d - b.d),
        })),
    }));

    let city = "";
    let chart;
    let ctx;

    // FOR RENDERING
    let properties = [];

    // FOR CHANGING VALUES
    let propertyTypes = [];

    onMount(() => {
        city = jsonData[0].city;
    });

    function getProperties(city) {
        return city.properties.map((prop) => prop.propertytype);
    }

    function getLabels(city) {
        return city.properties[1].data.map(
            (dataObj) => dataObj.d.toISOString().split("T")[0]
        );
    }

    function getData(city) {
        return city.properties.map((prop) => ({
            propertyType: prop.propertytype,
            data: prop.data.map((b) => b.b),
        }));
    }

    function getCityData(city) {
        return jsonData.find((doc) => doc.city === city);
    }

    function getCityChartData(city) {
        let cityProperties = getProperties(city);
        let cityData = getData(city);
        let cityLabels = getLabels(city);

        return {
            cityLabels,
            cityProperties,
            cityData,
        };
    }

    function drawChart(labels, dataArray) {
        let colors = ["#2ecc71", "#2980b9", "#8e44ad", "red", "#f1c40f"];
        let lightColors = [
            "#d5f5e2",
            "#d1e6f4",
            "#e9d8f0",
            "#d0dbe5",
            "#fcf3cf",
        ];

        const config = {
            type: "line",
            data: {
                labels: labels,

                datasets: dataArray.map((row, i) => {
                    return {
                        label: row.propertyType,
                        backgroundColor: lightColors[i],
                        fill: false,
                        borderColor: colors[i],
                        data: row.data,
                    };
                }),
            },
            options: {
                responsive: true,
                scales: {
                    yAxes: {
                        title: {
                            display: true,
                            text: "Value",
                            font: {
                                size: 15,
                            },
                        },
                        ticks: {
                            precision: 0,
                        },
                    },
                    xAxes: {
                        title: {
                            display: true,
                            text: "Date",
                            font: {
                                size: 15,
                            },
                        },
                    },
                },
            },
        };

        if (chart) chart.destroy();

        chart = new Chart(ctx.getContext("2d"), config);
    }

    function handleChange(city) {
        if (city) {
            let selectedCity = getCityData(city);

            let { cityLabels, cityProperties, cityData } =
                getCityChartData(selectedCity);

            console.log(cityData);

            properties = [...cityProperties];
            propertyTypes = [...cityProperties];

            drawChart(cityLabels, cityData);
        } else if (chart) {
            properties = [];
            chart.destroy();
        }
    }

    $: if (city) {
        handleChange(city);
    }

    $: if (propertyTypes.length) {
        let selectedCity = getCityData(city);
        if (selectedCity) {
            let { cityLabels, cityData } = getCityChartData(selectedCity);

            let copyData = [];
            cityData.forEach((row, i) => {
                if (propertyTypes.includes(row.propertyType)) {
                    copyData.push(row);
                }
            });

            drawChart(cityLabels, copyData);
        }
    }
</script>

<Header />
<main>
    <aside class="list-container">
        <h2>&mdash; Select City &mdash;</h2>

        <div class="radiocontainer">
            {#each jsonData as doc (doc.city)}
                <input
                    type="radio"
                    name="Selected City"
                    value={doc.city}
                    id={doc.city}
                    bind:group={city}
                />
                <label for={doc.city}>{doc.city.toUpperCase()}</label>
            {/each}
        </div>

        <!-- <select
            name="cityList"
            id="cityList"
            bind:value={city}
            on:change={handleChange}
        >
            <option value="">Select City</option>
            {#each jsonData as doc (doc.city)}
                <option value={doc.city}>{doc.city.toUpperCase()}</option>
            {/each}
        </select> -->
    </aside>

    <section class="chart-container">
        <!-- {#if city} -->
        <div class="city-details">
            <h2>Selected City: {city.toUpperCase()}</h2>

            <div class="city-properties">
                {#each properties as prop (prop)}
                    <div class="property">
                        <input
                            type="checkbox"
                            bind:group={propertyTypes}
                            value={prop}
                            id={prop}
                        />
                        <label for={prop}>{prop}</label>
                    </div>
                {/each}
            </div>
        </div>
        <canvas id="chart" bind:this={ctx} />
        <!-- {:else}
            <h2>Please Select a City</h2>
        {/if} -->
    </section>
</main>
<Footer />

<style>
    main {
        flex: 1;

        display: flex;
        gap: 3rem;
    }
    main > * {
        padding: 2rem;
    }
    main aside {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        border-right: 1px solid var(--color-asbestos);
    }
    main section {
        flex: 1;
    }

    /* ASIDE */
    h2 {
        font-size: 3rem;
        text-align: center;
        margin-bottom: 2rem;
    }
    /* select {
        width: 100%;
        border-radius: 0.5rem;
        padding: 0.6rem;
    }
    select option {
        display: inline-block;
        font-size: 1.6rem;
    } */

    .radiocontainer {
        display: grid;
        grid-template-columns: max-content 1fr;
        grid-gap: 1rem;
        align-items: center;
        font-size: 1.6rem;
    }

    /* Chart Container */
    .property,
    .city-details {
        display: flex;
        justify-content: space-between;
        align-items: center;
        gap: 1rem;
    }

    .city-properties {
        flex-direction: column;
    }
    .property label {
        font-size: 1.6rem;
    }
</style>
