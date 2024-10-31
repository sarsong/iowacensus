<script>
  import * as d3 from 'd3';
  import { onMount } from 'svelte';
  import Map from './Map.svelte';
  import Histogram from './Histogram.svelte';

  let data = [];
  let fullData = [];
  let filter1 = [];
  let filter2 = [];
  let filter3 = []

  let var1 = 'Median Income';
  let var2 = 'Percent White';
  let var3 = "Percent of Population over 25 with Bachelor's Degree or Higher"

  let selectedRace = var2;
  let selectedVariable = var1;

  let loading = true;

  onMount(async () => {
    // let table = await d3.csv('/iowa_percents.csv', (d) => {
    //   const row = { ...d };

    //   for (const key in row) {
    //     if (!isNaN(row[key]) && row[key] != '') {
    //       row[key] = +row[key];
    //     }
    //   }
    //   return row;
    // });

    // let geocoord = await d3.json('/tl_2023_19_tract.geojson')
    //   .then((d) => d.features);

    // geocoord.forEach((feature) => {
    //   const matchingRow = table.find((row) => row.GEO_ID == feature.properties.GEOIDFQ);
    //   if (matchingRow) {
    //     feature.properties = { ...feature.properties, ...matchingRow };
    //   }
    // });
    
    // data = geocoord; 
    // fullData = [...data];

    let table = d3.csv('/iowa_percents.csv', (d) => {
      const row = { ...d };
      for (const key in row) {
        if (!isNaN(row[key]) && row[key] != '') {
          row[key] = +row[key];
        }
      }
      return row;
      
    });

    let geocoord = d3.json('/tl_2023_19_tract.geojson').then((d) => d.features);

    await Promise.all([table, geocoord]).then(([table, geocoord]) => {
      geocoord.forEach((feature) => {
        const matchingRow = table.find((row) => row.GEO_ID == feature.properties.GEOIDFQ);
        if (matchingRow) {
          feature.properties = { ...feature.properties, ...matchingRow };
        }
      });

      data = geocoord;
      fullData = [...geocoord];
      loading = false;
    });
    console.log(data); 
});

  function updateData() {
    if (filter1.length > 0 && filter2.length > 0 && filter3.length > 0) {
        data = fullData.filter((d) => (d.properties[var1] >= filter1[0] && d.properties[var1] < filter1[1] && d.properties[selectedRace] >= filter2[0] && d.properties[selectedRace] < filter2[1]  && d.properties[var3] >= filter3[0] && d.properties[var3] < filter3[1]));
    } else if (filter1.length > 0 && filter2.length == 0 && filter3.length == 0) {
        data = fullData.filter((d) => (d.properties[var1] >= filter1[0] && d.properties[var1] < filter1[1]));
    } else if (filter1.length == 0 && filter2.length > 0 && filter3.length == 0) {
        data = fullData.filter((d) => (d.properties[selectedRace] >= filter2[0] && d.properties[selectedRace] < filter2[1]));
    } else if (filter1.length == 0 && filter2.length == 0 && filter3.length > 0) {
        data = fullData.filter((d) => (d.properties[var3] >= filter3[0] && d.properties[var3] < filter3[1]));
    } else {
      data = [...fullData];
    }
  }
</script>

<main>
  <h1>Iowa Census Data</h1>

  <div class="flex-container row">
    <div class="flex-container col">
      <label for="race-select">Select Map Variable: </label>
          <select id="race-select" bind:value={selectedVariable}>
            <option value="Median Income">Median Income</option>
            <option value="Percent of Population over 25 with Bachelor's Degree or Higher">Percent of Population over 25 with Bachelor's Degree or Higher</option>
            <option value="Percent White">White</option>
            <option value="Percent Black or African American">Black or African American</option>
            <option value="Percent Asian">Asian</option>
            <option value="Percent American Indian and Alaska Native">American Indian and Alaska Native</option>
            <option value="Percent Native Hawaiian and Other Pacific Islander">Native Hawaiian and Other Pacific Islander</option>
            <option value="Percent Hispanic or Latino">Hispanic or Latino</option>
            <option value="Percent Other Race">Other Race</option>
            <option value="Percent Two or More Races">Two or More Races</option>
            <option value="Median Age">Median Age</option>
          </select>
      <div class="map">
        <Map data={data} fullData={fullData} variable={selectedVariable}></Map>
      </div>
    </div>

    <div class="flex-container col">
      <label for="race-select">Select Histogram Race: </label>
        <select id="race-select" bind:value={selectedRace}>
          <option value="Percent White">White</option>
          <option value="Percent Black or African American">Black or African American</option>
          <option value="Percent Asian">Asian</option>
          <option value="Percent American Indian and Alaska Native">American Indian and Alaska Native</option>
          <option value="Percent Native Hawaiian and Other Pacific Islander">Native Hawaiian and Other Pacific Islander</option>
          <option value="Percent Hispanic or Latino">Hispanic or Latino</option>
          <option value="Percent Other Race">Other Race</option>
          <option value="Percent Two or More Races">Two or More Races</option>
        </select>
      <div class="hist">
        <Histogram data={data} fullData={fullData} variable={selectedRace} bind:filter={filter2} update={updateData}></Histogram>
      </div>
      <div class="hist">
        <Histogram data={data} fullData={fullData} variable={var1} bind:filter={filter1} update={updateData}></Histogram>
      </div>
      <div class="hist">
        <Histogram data={data} fullData={fullData} variable={var3} bind:filter={filter3} update={updateData}></Histogram>
      </div>
    </div>
  </div>
</main>

<style>
  .flex-container {
    display: flex; 
    justify-content: center;  
    height: 100%; 
    padding: 15px; 
    gap: 5px;
  }
    
  .flex-container > div {
    padding: 8px;
  }

  .flex-container .row {
    flex-direction: row;  
  }

  .flex-container .col {
    flex-direction: column;  
  }

  .map { 
    flex-grow: 1;
  }

  .hist {
    flex-grow: 0.5;
  }
</style>
