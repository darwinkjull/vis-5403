<script>
	import * as d3 from 'd3';
	import { onMount } from 'svelte';
  
	let data = [];
	let selectedCondition = "Acute Coronary Syndrome";
  
	onMount(async () => {
	  try {
		const importData = await d3.csv("/patient_data.csv");
		data = importData.map(row => ({
		  Age: +row.Age,
		  Sex: row.Sex || "",
		  [selectedCondition]: +(row[selectedCondition] || 0),
		})).filter(d => !isNaN(d.Age));
		createVisualization();
	  } catch (error) {
		console.error("Error loading CSV:", error);
	  }
	});
  
	function createVisualization() {
	  const svgWidth = 600, svgHeight = 400;
	  const margin = { top: 20, right: 20, bottom: 30, left: 40 };
	  const width = svgWidth - margin.left - margin.right;
	  const height = svgHeight - margin.top - margin.bottom;
  
	  const svg = d3.select("svg").attr("viewBox", `0 0 ${svgWidth} ${svgHeight}`);
  
	  svg.selectAll("*").remove(); // Clear any existing visualization
  
	  const x = d3.scaleLinear()
		.domain([0, d3.max(data, d => d.Age) || 0])
		.range([margin.left, width - margin.right]);
  
	  const histogram = d3.histogram()
		.domain(x.domain())
		.thresholds(x.ticks(20));
  
	  const bins = histogram(data.map(d => d.Age));
  
	  const y = d3.scaleLinear()
		.domain([0, d3.max(bins, d => d.length) || 0])
		.range([height, margin.top]);
  
	  svg.selectAll(".bar")
		.data(bins)
		.enter().append("rect")
		.attr("class", "bar")
		.attr("x", d => x(d.x0))
		.attr("y", d => y(d.length))
		.attr("width", d => x(d.x1) - x(d.x0) - 1)
		.attr("height", d => height - y(d.length))
		.attr("fill", "steelblue");
  
	  svg.append("g")
		.attr("transform", `translate(0,${height})`)
		.call(d3.axisBottom(x));
  
	  svg.append("g")
		.attr("transform", `translate(${margin.left},0)`)
		.call(d3.axisLeft(y));
	}
  </script>
  
  <main>
	<h1>D3 Visualization</h1>
	<select bind:value={selectedCondition}>
	  <option value="Acute Coronary Syndrome">Acute Coronary Syndrome</option>
	  <option value="Angina">Angina</option>
	  <option value="Arrhythmias">Arrhythmias</option>
	  <option value="Atrial Fibrillation">Atrial Fibrillation</option>
	</select>
	<svg width="600" height="400"></svg>
  </main>
  
  <style>
	main {
	  text-align: center;
	  padding: 1em;
	}
  
	select {
	  margin-bottom: 1em;
	}
  </style>
  