<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let sliderYear = 2000; 
  let sliderLabel = `Year: ${sliderYear}`;
  let svg; 
  let allData = {}; 
  let hoverState = null; 
  
  $: sliderLabel = `Year: ${sliderYear}`;

  const colorScale = d3.scaleSequential(d3.interpolateBlues);

  onMount(async () => {
    svg = d3.select("#map").append("svg")
      .attr("width", 960)
      .attr("height", 600);

    const projection = d3.geoAlbersUsa().translate([480, 300]).scale(1000);
    const path = d3.geoPath().projection(projection);

    try {
      const geoData = await d3.json("Simple.geojson");
      console.log("GeoJSON loaded successfully", geoData);
    } catch (error) {
      console.error("Failed to load GeoJSON", error);
    }
    const nestedData = await d3.json("nested_data.json");
    allData = nestedData;
    
    const paths = svg.selectAll("path")
      .data(geoData.features)
      .enter().append("path")
      .attr("d", path)
      .attr("fill", d => {
        const stateName = d.properties.NAME;
        const value = nestedData[sliderYear][stateName] || 0;
        return colorScale(value);
      })
      .attr("stroke", "white")
      .attr("stroke-width", "1.5")
      .on("mouseover", (event, d) => {
        hoverState = { name: d.properties.NAME, value: allData[sliderYear][d.properties.NAME] || 0 };
        d3.select(event.currentTarget).style("fill", "orange");
      })
      .on("mouseout", (event, d) => {
        hoverState = null;
        d3.select(event.currentTarget).style("fill", d => {
          const stateName = d.properties.NAME;
          const value = allData[sliderYear][stateName] || 0;
          return colorScale(value);
        });
      });

    const legendWidth = 400;
    const legendHeight = 30;
    const legendMargin = { top: 20, right: 20, bottom: 0, left: 20 };

    const legendSvg = svg.append("g")
      .attr("transform", `translate(${(svg.attr("width") - legendWidth) / 2},${svg.attr("height") - legendMargin.bottom - legendHeight})`);

    const legendScale = d3.scaleLinear()
      .domain([0, .8]) 
      .range([0, legendWidth]);

    const legendAxis = d3.axisBottom(legendScale)
      .ticks(0) 
      .tickSize(0)
      .tickFormat(""); 

    legendSvg.append("g")
      .attr("class", "legend-axis")
      .call(legendAxis)
      .select(".domain")
      .remove(); 

    const gradient = legendSvg.append("defs")
      .append("linearGradient")
      .attr("id", "gradient")
      .attr("x1", "0%")
      .attr("x2", "100%")
      .attr("y1", "0%")
      .attr("y2", "0%");

    gradient.append("stop")
      .attr("offset", "0%")
      .attr("stop-color", colorScale(0));

    gradient.append("stop")
      .attr("offset", "100%")
      .attr("stop-color", colorScale(1));

    legendSvg.append("rect")
      .attr("width", legendWidth)
      .attr("height", legendHeight)
      .style("fill", "url(#gradient)");

    legendSvg.append("text")
      .attr("class", "legend-label")
      .attr("x", 0)
      .attr("y", -5)
      .attr("text-anchor", "start")
      .text("Lower Income (Relative)");

    legendSvg.append("text")
      .attr("class", "legend-label")
      .attr("x", legendWidth)
      .attr("y", -5)
      .attr("text-anchor", "end")
      .text("Higher Income (Relative)");
  });

  $: {
    if (svg && allData[sliderYear]) {
      updateColorsAndFill();
    }
  }

  function updateColorsAndFill() {
    const incomes = Object.values(allData[sliderYear]).filter(Boolean);
    const minIncome = Math.min(...incomes);
    const maxIncome = Math.max(...incomes);
    colorScale.domain([minIncome, maxIncome]);

    svg.selectAll("path")
      .attr("fill", d => {
        const stateName = d.properties.NAME;
        const value = allData[sliderYear][stateName] || 0;
        return colorScale(value);
      });
  }
</script>

<main>
  <div class="title-container">
    <h1>Median Household Income Distribution</h1>
  </div>
  <div class="overlay">
    <label for="slider">{sliderLabel}</label>
    <input
      id="slider"
      type="range"
      min="1984"
      max="2018"
      bind:value={sliderYear}
    />
  </div>
  {#if hoverState}
  <div class="tooltip">
    {hoverState.name}: ${hoverState.value.toLocaleString()}
  </div>
  {/if}
</main>

<style>
</style>