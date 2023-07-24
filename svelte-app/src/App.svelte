<script>
		import { geoPath, geoTransverseMercator } from 'd3-geo';
    import json from './Mississippi-Counties.json';
		import * as d3 from 'd3';


		let title = "(Test) Total Median Earnings By County In Mississippi";
		let legend;
    let width = 200;
    $: height = width * 1.5;
    $: projection = geoTransverseMercator()
        .rotate([90,0])
        .fitExtent([[10, 10], [width, height]], json)
		    // .translate([width / 2, height / 2]);

    $: path = geoPath().projection(projection);
    const counties = json.features;

		import data from "./Data_Test_2023_07_10.json"; // https://data.worldbank.org/indicator/SP.POP.TOTL
	  // Restructure counties array to include Total Median Earnings
		// console.log(counties)
	  counties.forEach(county => {
			// console.log("county.prop")
			// console.log(county.properties.GEOID)
	    const metadata = data.find(d => d.Id2 == county.properties.GEOID);
	    if (metadata) {
				// console.log("metadata")
				// console.log(metadata)
	      county.earnings = metadata.Total_Median_Earnings;
	      county.county = metadata.Geography;
				// console.log("earnings: " + county.earnings)
				// console.log(county.county)
	    }
	  });

	// Color scale
  import { max } from "d3-array";
	import { schemeBlues } from 'd3';
  import { scaleLinear, scaleQuantile } from "d3-scale";
	// import {Legend} from "d3/color-legend";
	import {legendColor} from 'd3-svg-legend';

  const colorScale1 = scaleLinear()
    .domain([0, max(data, d => d.Total_Median_Earnings)])
    .range(["#ebc5e4", "#57204e"]);

	// using blue color scheme from: https://observablehq.com/@d3/color-schemes
	const colorScale2 = scaleQuantile()
	.domain([0, max(data, d => d.Total_Median_Earnings)])
	.range(["#eff3ff","#bdd7e7","#6baed6","#3182bd","#08519c"]);

	// https://d3-legend.susielu.com/#color-quant ?? 
	// const colorLegend = legendColor()
	// 		.shape('circle')
	// 		.scale(colorScale2)

	// 	d3.select(legend)
	// 		.call(colorLegend);

	// make another component to build a legend
		let hoveredData;
</script>
<h1 class="center">{title}</h1>
<div class="svg-container" bind:clientWidth={width}>
    <svg {width} {height}>
        <!-- <circle cx="{width/2}" cy="{height/2}" r="2" fill="blue"></circle> -->
				{#each counties as county, i}
						<path 
							d={path(county)} 
							fill={colorScale2(county.earnings || 0)} 
							stroke={hoveredData === county? "yellow": "black"}
							stroke-width={hoveredData === county? 2: 0.5}
							on:mouseover={() => hoveredData = county}
					    on:mouseleave={() => hoveredData = null}
							on:focus={() => hoveredData = county}
							/>
				{/each}
			<!-- <legend fill={colorLegend}/> -->
			<!-- <g bind:this={legend} /> -->
    </svg>
</div>

	
<style>
    .center {
        text-align: center;
    }
    
    .svg-container {
        /* width: 100%; */
        /* height: 100%; */
        /* resize: both; */
        /* overflow: auto; */
				/* to show the svg contianer */
				border: 1px dashed #aaa;
				position: relative;
		    max-width: 400px;
				max-height: 600px;
		    margin: 0 auto;
				padding-bottom: 10px;
    }
		svg {
	    overflow: visible;
	  }
	
	  path {
	    cursor: pointer;
	  }

		h1 {
	    font-size: 1.5rem;
	    font-weight: 600;
	    margin-bottom: 0.35rem;
			text-align: center;
	  }

</style>