<main>
  <div class='main'>
    <h1 style="font-size: 2rem; font-weight: bold;">How does median income vary across the US?</h1>
    <div id="myDiv">
      <h1 style="text-align: center;  font-weight: bold;">Average Income of Any Type of Households By State</h1>
    </div>
    
  </div>
</main>

<svelte:head>
  <link href="https://cdn.jsdelivr.net/npm/daisyui@2.11.0/dist/full.css" rel="stylesheet" type="text/css" />
  <script src="https://cdn.tailwindcss.com"></script>
</svelte:head>

<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import * as topojson from 'topojson-client';
  import { Legend } from '../helpers'

  let svgElement;
  let legend;
  let incomeData = new Map();
  let domain = [];

  let isDropdownOpen = false // default state (dropdown close)
  let name = "Households";
  let count = 0;

  const stateCodes = {
  "Alabama": "AL",
  "Alaska": "AK",
  "Arizona": "AZ",
  "Arkansas": "AR",
  "California": "CA",
  "Colorado": "CO",
  "Connecticut": "CT",
  "Delaware": "DE",
  "District of Columbia": "DC",
  "Florida": "FL",
  "Georgia": "GA",
  "Hawaii": "HI",
  "Idaho": "ID",
  "Illinois": "IL",
  "Indiana": "IN",
  "Iowa": "IA",
  "Kansas": "KS",
  "Kentucky": "KY",
  "Louisiana": "LA",
  "Maine": "ME",
  "Maryland": "MD",
  "Massachusetts": "MA",
  "Michigan": "MI",
  "Minnesota": "MN",
  "Mississippi": "MS",
  "Missouri": "MO",
  "Montana": "MT",
  "Nebraska": "NE",
  "Nevada": "NV",
  "New Hampshire": "NH",
  "New Jersey": "NJ",
  "New Mexico": "NM",
  "New York": "NY",
  "North Carolina": "NC",
  "North Dakota": "ND",
  "Ohio": "OH",
  "Oklahoma": "OK",
  "Oregon": "OR",
  "Pennsylvania": "PA",
  "Puerto Rico": "PR",
  "Rhode Island": "RI",
  "South Carolina": "SC",
  "South Dakota": "SD",
  "Tennessee": "TN",
  "Texas": "TX",
  "Utah": "UT",
  "Vermont": "VT",
  "Virginia": "VA",
  "Washington": "WA",
  "West Virginia": "WV",
  "Wisconsin": "WI",
  "Wyoming": "WY"
} //not being used rn, would be if we can get labels to show on zoom

  const handleDropdownClick = () => {
    isDropdownOpen = !isDropdownOpen // togle state on click
  }

  const handleDropdownFocusLoss = ({ relatedTarget, currentTarget }) => {
    if (relatedTarget instanceof HTMLElement && currentTarget.contains(relatedTarget)) return 
  }
  async function fetchData(name) {
    const us = await d3.json("/states-albers-10m.json");
    const csvData = await d3.csv("/census_cleaned.csv");
    csvData.filter(d => d.Category === name).forEach(d => {
      incomeData.set(d.State, d.median_income);
    });
    const asArray = [...incomeData].map(([name, value]) => (Number(value)));
    domain = d3.extent(asArray);
    initializeChart(us);
    createLegend();
    
  }
  onMount(() => {

    fetchData(name);
  });
  //This function will let the user know what data they are looking at also gets ride of extra legends
  function changeContent() {
    if (name == "Families") {
      var sdiv = document.getElementById('myDiv');
      var removeLegend = document.getElementById('legend-container');
      sdiv.innerHTML = '<h1 style="text-align: center; font-weight: bold;">Average Income of Households of Any Family Type By State</h1>';
      removeLegend.innerHTML = '';
    }
    else if (name == "Households") {
      var sdiv = document.getElementById('myDiv');
      var removeLegend = document.getElementById('legend-container');
      sdiv.innerHTML = '<h1 style="text-align: center; font-weight: bold;">Average Income of Any Type of Households By State</h1>';
      removeLegend.innerHTML = '';
    }
    else if (name == "Married-couple families") {
      var sdiv = document.getElementById('myDiv');
      var removeLegend = document.getElementById('legend-container');
      sdiv.innerHTML = '<h1 style="text-align: center; font-weight: bold;">Average Income of Households of the Married-couple Family Type By State</h1>';
      removeLegend.innerHTML = '';
    }
    else if (name == "Nonfamily households") {
      var sdiv = document.getElementById('myDiv');
      var removeLegend = document.getElementById('legend-container');
      sdiv.innerHTML = '<h1 style="text-align: center; font-weight: bold;">Average Income of Non-Family Households By State</h1>';
      removeLegend.innerHTML = '';
    }
    console.log(name)
}
  //For some reason making separate functions for each button click works best to get the data to update
  function handleClickFam() {
    name = "Families"
    changeContent()
    fetchData(name)
    
  }
	function handleClickHouse() {    
    name = "Households"
    changeContent()
    fetchData(name)    
  }
  function handleClickMarried() {
    name = "Married-couple families"
    changeContent()
    fetchData(name)
  }
  /*function handleMouseover(event) {
    const state = event.target.getAttribute("data-state");
    const income = incomeData.get(state);
    hoverData = { state, income };
    
	}*/
  function handleClickNonFam() {
    name = "Nonfamily households"
    changeContent()
    fetchData(name)
  }  

  function initializeChart(us) {
    const width = 1175;
    const height = 810;
    const colorScale = d3.scaleLinear()
      .domain(domain) //d3.extent(Array.from(incomeData.values()))
      .range(["lightgreen", "darkgreen"]); // Updated color scale from light green to dark green

    const zoom = d3.zoom()
    .scaleExtent([1, 8])
    .on("zoom", zoomed);

    const svg = d3.select(svgElement)
        .attr("viewBox", `0 0 ${width} ${height}`)
        .attr("preserveAspectRatio", "xMidYMid meet")
        .attr("width", width)
        .attr("height", height)
        .style("display", "block")
        .style("margin", "auto")
        .style("overflow", "hidden")
        .on('click', reset)

    const path = d3.geoPath();
    const g = svg.append("g");

    let states = g.selectAll("path")
      .data(topojson.feature(us, us.objects.states).features)
      .join("path")
      .on('click', clicked)
      .attr('d', path)
      .attr("fill", d => {
        const income = incomeData.get(d.properties.name);
        return income ? colorScale(income) : "#cccccc";
      })
      .attr("d", path)
      .append("title")
      .text(d => `${d.properties.name}: $${Number(incomeData.get(d.properties.name)).toLocaleString() || 'No data'}`).exit()

    // const labels = g.selectAll('.label')
    //   .data(topojson.feature(us, us.objects.states).features)
    //     .enter()
    //   ;

    g.append("path")
      .attr("fill", "none")
      .attr("stroke", "white")
      .attr("stroke-linejoin", "round")
      .attr("d", path(topojson.mesh(us, us.objects.states, (a, b) => a !== b)));

    states = states.enter()
      .append('text')
        .attr("class", "label")
        .attr("transform", d => "translate(" + path.centroid(d) + ")")
        .text(d => `${stateCodes[d.properties.name]}\n $${Number(incomeData.get(d.properties.name)).toLocaleString()}`)
    const labels = g.selectAll('.label')
      .data(topojson.feature(us, us.objects.states).features)
        .enter()


    
    svg.call(zoom);

    let lastClicked;

    function clicked(event, d) {
      const [[x0, y0], [x1, y1]] = path.bounds(d);
      console.log(lastClicked);
      if (d == lastClicked) {
        reset();
        lastClicked = undefined
        return;
      }
      event.stopPropagation();

      svg.transition().duration(750).call(
        zoom.transform,
        d3.zoomIdentity
          .translate(width / 2, height / 2)
          .scale(Math.min(8, 0.9 / Math.max((x1 - x0) / width, (y1 - y0) / height)))
          .translate(-(x0 + x1) / 2, -(y0 + y1) / 2),
        d3.pointer(event, svg.node())
      );
      lastClicked = d;
    }

    function reset() {
      svg.transition().duration(750).call(
      zoom.transform,
      d3.zoomIdentity,
      d3.zoomTransform(svg.node()).invert([width / 2, height / 2])
    );
    }

    function zoomed(event) {
      const {transform} = event;
      g.attr("transform", transform);
      g.attr("stroke-width", 1 / transform.k);
    }
  }

  function createLegend() {
    const legendContainer = d3.select('#legend-container');
    const legend = Legend(d3.scaleLinear(domain, ['lightgreen', 'darkgreen']), { title: 'Median Income (USD)', ticks: 5, tickFormat: (d) => '$' + d.toLocaleString() });
      legendContainer
        .append(() => legend);
  }


  
  
</script>


<div class="flex justify-between items-center">
	<div class="dropdown" on:focusout={handleDropdownFocusLoss}>
    <button class="btn m-4 w-52" style="background-color: #013220;" on:click={handleDropdownClick}>    
		{#if isDropdownOpen}
			<svg
								fill="none"
								viewBox="0 0 24 24"
								class="inline-block h-6 w-6 stroke-current">
								<title>Close Dropdown</title>
								<path
									stroke-linecap="square"
									stroke-linejoin="square"
									stroke-width="5"
									d="M6 18L18 6M6 6l12 12" />
							</svg>
			{:else}
      Filter by Household Type
			
			{/if}
		</button>
		<ul class="dropdown-content menu p-2 shadow bg-base-100 rounded-box w-52 m-4" style:visibility={isDropdownOpen ? 'visible' : 'hidden'}>
			<li><button on:click={handleClickHouse}>Households of Any Type </button></li>
			<li><button on:click={handleClickFam}>Households of Any Family Type </button></li>
      <li><button on:click={handleClickMarried}>Households of Married-couple Family Type</button></li>
      <li><button on:click={handleClickNonFam}>Households of Nonfamily Type</button></li>

      
		</ul>
	</div>
	
</div>




<svelte:element this="svg" bind:this={svgElement} />

<div class=legend id=legend-container/>

