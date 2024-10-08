<script>
  import { onMount } from 'svelte';

  let tableWrapper, table, tableCells, tooltip;

  const hoverColor = '#FEF1EF';

  export let categories;
  export let currentTableData;

  const sandboxBaseURL = 'https://hapi.humdata.org/docs#';
  const sandboxURL = {
    'humanitarian-needs': `${sandboxBaseURL}/Affected%20people/get_humanitarian_needs_api_v1_affected_people_humanitarian_needs_get`,
    'idps': `${sandboxBaseURL}/Affected%20people/get_idps_api_v1_affected_people_idps_get`,
    'refugees': `${sandboxBaseURL}/Affected%20people/get_refugees_api_v1_affected_people_refugees_get`,
    'returnees': `${sandboxBaseURL}/Affected%20people/get_returnees_api_v1_affected_people_returnees_get`,
    'conflict-event': `${sandboxBaseURL}/Coordination%20%26%20Context/get_conflict_events_api_v1_coordination_context_conflict_event_get`,
    'funding': `${sandboxBaseURL}/Coordination%20%26%20Context/get_fundings_api_v1_coordination_context_funding_get`,
    'national-risk': `${sandboxBaseURL}/Coordination%20%26%20Context/get_national_risks_api_v1_coordination_context_national_risk_get`,
    'operational-presence': `${sandboxBaseURL}/Coordination%20%26%20Context/get_operational_presences_api_v1_coordination_context_operational_presence_get`,
    'food-price': `${sandboxBaseURL}/Food%20Security%20%26%20Nutrition/get_food_prices_api_v1_food_food_price_get`,
    'food-security': `${sandboxBaseURL}/Food%20Security%20%26%20Nutrition/get_food_security_api_v1_food_food_security_get`,
    'population': `${sandboxBaseURL}/Population%20%26%20Socio-Economy/get_populations_api_v1_population_social_population_get`,
    'poverty-rate': `${sandboxBaseURL}/Population%20%26%20Socio-Economy/get_poverty_rates_api_v1_population_social_poverty_rate_get`
  }

  function onMouseover(e) {
    const cell = event.target;
    if (cell.classList.contains('cell') && !cell.classList.contains('fixed-col') && cell.tagName !== 'TH') {
      highlightCells(cell);
      //showTooltip(cell);
    }
    else {
      //hideTooltip();
    }
  }

  function onMouseout() {
    resetCells();
  }

  function highlightCells(cell) {
    // highlight hovered cell and cells to left and above
    tableCells = document.querySelectorAll('#coverageTable td, #coverageTable th');
    const row = cell.parentElement;

    // get current cell position
    const rowIndex = Array.from(table.rows).indexOf(row);
    const cellIndex = Array.from(row.cells).indexOf(cell);

    // highlight cells above current cell in same column
    for (let i = 1; i <= rowIndex; i++) {
      table.rows[i].cells[cellIndex].style.backgroundColor = hoverColor;
    }

    // highlight cells to the left in same row
    for (let i=0; i <=cellIndex; i++) {
      row.cells[i].style.backgroundColor = hoverColor;
    }
  }

  function resetCells() {
    tableCells.forEach(cell => {
      cell.style.backgroundColor = '';
    });
  }

  function showTooltip(cell) {
    // get dimensions of hovered cell and parent table
    const tableRect = table.getBoundingClientRect();
    const rect = cell.getBoundingClientRect();

    // get position of hovered cell and parent table
    const xPos = rect.left - tableRect.left + rect.width - 50;
    const yPos = rect.top - tableRect.top - 15;

    tooltip.style.opacity = 1;
    tooltip.style.left = xPos + 'px';
    tooltip.style.top = yPos + 'px';

    tooltip.innerHTML = 'Administrative boundaries: <b>0–2</b><br>';
    tooltip.innerHTML += 'Data provider: <b>OCHA HPC</b><br>';
    tooltip.innerHTML += 'Date range: <b>Jan 1, 2024–Dec 31, 2024</b><br>';
    tooltip.innerHTML += 'Update frequency: <b>Anually</b><br>';
  }

  function hideTooltip() {
    tooltip.style.opacity = 0;
  }

  function setTableHeight(table) {
    const yPos = table.getBoundingClientRect().top;
    const availHeight = window.innerHeight - yPos;
    table.style.height = `${availHeight}px`;
  }

  function formatStr(str) {
    return str
      .split('-')
      .map(word => word.charAt(0).toUpperCase() + word.slice(1))
      .join(' ');
  }

  onMount(async () => {
    table = document.getElementById('coverageTable');

    // set table height equal to available screen height
    tableWrapper = document.querySelector('.table-wrapper');
    setTableHeight(tableWrapper);

    // create tooltip
    tooltip = document.querySelector('.tooltip');

    // add event listener for the scroll event
    // tableWrapper.addEventListener('scroll', (event) => {
    //   tooltip.style.opacity = 0;
    // });
  });
</script>

<div class='container'>
  <div class='table-wrapper'>
    <table id='coverageTable'>
      <thead>
        <tr>
          <th class='fixed-col category'></th>
          {#each Object.entries(categories) as [category, subcategories]}
            <th class='category' colspan={subcategories.length}>{formatStr(category)}</th>
          {/each}
        </tr>
        <tr>
          <th class='fixed-col'></th>
          {#each Object.entries(categories) as [category, subcategories]}
            {#each subcategories as subcategory}
              <th>{formatStr(subcategory)}<br>
                <span>
                  <a href={(sandboxURL[subcategory] !== undefined) ? sandboxURL[subcategory] : 'https://hapi.humdata.org/docs#/'} target='_blank'>Go to API Sandbox</a>
                </span>
              </th>
            {/each}
          {/each}
        </tr>
      </thead>
      <tbody>
        {#each Object.entries(currentTableData) as [country, subcategories]}
          <tr>
            <td class='fixed-col'><div class='country'>{country}</div></td>
            {#each Object.entries(subcategories) as [subcategory, hasData]}
              <td class='cell' on:mouseover={onMouseover} on:mouseout={onMouseout}>
                <div class='admin-key ${subcategory}'>
                  {#if (!hasData.admin0 && !hasData.admin1 && !hasData.admin2)}
                    <div><i class='no-data'></i></div>
                  {:else}
                    <div class={`admin-icon national ${hasData.admin0 ? '' : 'hide'}`}>0</div>
                    <div class={`admin-icon subnational1 ${hasData.admin1 ? '' : 'hide'}`}>1</div> 
                    <div class={`admin-icon subnational2 ${hasData.admin2 ? '' : 'hide'}`}>2</div>
                  {/if}
                </div>
              </td>
            {/each}
          </tr>
        {/each}
      </tbody>
    </table>
    
    <div class='tooltip'>tooltip here</div>
  </div>

  <div class='gradient-overlay'></div>
</div>


<style lang='scss'>
  .container {
    position: relative;
  }
  .table-wrapper {
    display: block;
    height: 600px;
    margin: 0;
    overflow: auto;
    padding: 0;
    position: relative;
    width: 100%;
  }
  table {
    border: 0;
    border-collapse: separate;
    border-spacing: 0;
    margin-right: 50px;
    width: 100%;
  }

  th, td {
    border-right: 1px solid #E6E9F5;
    line-height: 16px;
    padding: 8px 12px;
    &:last-child {
      border-right: 0;
    }
  }

  td {
    border-top: 1px solid #E6E9F5;
  }

  thead th {
    background-color: #FFF;
    font-family: 'Gotham-Bold';
    font-size: 14px;
    line-height: 16px;
    position: sticky;
    padding: 10px 12px;
    top: 38px;
  }

  th span {
    display: block;
    font-family: 'Source Sans 3', sans-serif;
    font-size: 14px;
    font-weight: 400;
    padding: 4px 0;
  }

  th.category {
    background-color: #EEE;
    border: 1px solid #FFF;
    top: 0;
    &:nth-child(1) {
      background-color: #FFF;
    }
    &:nth-child(2) {
      border-left: 0;
    }
  }

  th.fixed-col.category,
  .fixed-col {
    background-color: #FFF;
    font-family: 'Gotham-Bold';
    font-size: 14px;
    line-height: 18px;
    left: 0;
    position: -webkit-sticky;
    position: sticky;
    text-transform: uppercase;
    width: 250px;
    z-index: 3;
  }
  .fixed-col .country {
    padding: 10px 12px;
    width: 250px;
  }

  thead .fixed-col {
    border-top: 0;
  }

  tbody .fixed-col {
    z-index: 1;
  }

  .gradient-overlay {
    background: linear-gradient(to right, rgba(255, 255, 255, 0) 0%, rgba(255, 255, 255, 1) 100%);
    bottom: 0;
    right: 0;
    top: 0;
    pointer-events: none;
    position: absolute;
    width: 100px;
  }

  .tooltip {
    background-color: #FFF;
    border-radius: 8px;
    box-shadow: 0 0 8px rgba(0,0,0,.3);
    font-family: 'Source Sans 3', sans-serif;
    font-size: 14px;
    font-weight: normal;
    left: 0;
    line-height: 18px;
    padding: 8px 12px;
    pointer-events: none;
    position: absolute;
    opacity: 0;
    top: 0;
    width: auto;
    white-space: nowrap;
    z-index: 4;
    &::before {
      border-width: 8px;
      border-style: solid;
      border-color: transparent #FFF transparent transparent;
      content: '';
      margin-top: -8px;
      position: absolute;
      right: 100%;
      top: 50%;
    }
  }
</style>