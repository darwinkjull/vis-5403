<script>
  import * as d3 from "d3";

  let data = [];
  export let selectedCondition1;
  export let selectedCondition2;
  export let selectedScale;
  export let selectedSide; // Options: "same" or "opposite"
  export let enableCondition2; // Toggles the second histogram

  const MALE_COLOR = "#FFB000"; // Male bar color
  const FEMALE_COLOR = "#648FFF"; // Female bar color
  const MALE_COLOR_LIGHT = "#FE6100"; // Light version of male color
  const FEMALE_COLOR_LIGHT = "#785EF0"; // Light version of female color

  // Load CSV Data
  d3.csv("/patient_data.csv")
    .then(function (importData) {
      data = importData.map(d => ({
        "Sex": d.Sex,
        Age: +d.Age,
        "Acute Coronary Syndrome": +d["Acute Coronary Syndrome"],
        "Angina": +d["Angina"],
        "Arrhythmias": +d["Arrhythmias"],
        "Atrial Fibrillation": +d["Atrial Fibrillation"],
        "Cardiac Amyloidosis": +d["Cardiac Amyloidosis"],
        "Cardiomyopathy": +d["Cardiomyopathy"],
      }));
      console.log(data);
    })
    .catch(function (error) {
      console.error("Error loading CSV:", error);
    });

    
  // Reactive Data for Visualization
  // Reactive Data for Visualization
  $: selectedDataMale = data.length > 0 
    ? data.filter(d => d[selectedCondition1] === 1 && d.Sex === "Male")
          .map(d => d.Age)
          .filter(age => !isNaN(age)) 
    : [];
    
  $: selectedDataFemale = data.length > 0 
    ? data.filter(d => d[selectedCondition1] === 1 && d.Sex === "Female")
          .map(d => d.Age)
          .filter(age => !isNaN(age)) 
    : [];

  $: selectedDataMaleCondition2 = enableCondition2 && data.length > 0 
    ? data.filter(d => d[selectedCondition2] === 1 && d.Sex === "Male")
          .map(d => d.Age)
          .filter(age => !isNaN(age)) 
    : [];

  $: selectedDataFemaleCondition2 = enableCondition2 && data.length > 0 
    ? data.filter(d => d[selectedCondition2] === 1 && d.Sex === "Female")
          .map(d => d.Age)
          .filter(age => !isNaN(age)) 
    : [];

  // Trigger Visualization
  $: if ((selectedDataMale.length > 0 && selectedDataFemale.length > 0) && (selectedDataMaleCondition2.length > 0 && selectedDataFemaleCondition2.length > 0) && selectedSide || selectedScale || enableCondition2)  {
    createVisualization();
  }

// Create the D3 Visualization
function createVisualization() {
  const svgWidth = 800, svgHeight = 500;
  const margin = { top: 40, right: 40, bottom: 60, left: 60 };
  const width = svgWidth - margin.left - margin.right;
  const height = svgHeight - margin.top - margin.bottom;

  const svg = d3.select("#visualization")
    .attr("viewBox", `0 0 ${svgWidth} ${svgHeight}`)
    .attr("preserveAspectRatio", "xMidYMid meet");

  svg.selectAll("*").remove(); // Clear previous chart

  const x = d3.scaleLinear()
    .domain([0, 100]) // Fixed range for Age
    .range([margin.left, width - margin.right]);

  const histogram = d3.histogram()
    .domain(x.domain())
    .thresholds(x.ticks(20));

    const binsMale = histogram(selectedDataMale);
    const binsFemale = histogram(selectedDataFemale);
    const binsMaleCondition2 = histogram(selectedDataMaleCondition2);
    const binsFemaleCondition2 = histogram(selectedDataFemaleCondition2);

    const totalMale = selectedDataMale.length;
    const totalFemale = selectedDataFemale.length;
    const totalMaleCondition2 = selectedDataMaleCondition2.length;
    const totalFemaleCondition2 = selectedDataFemaleCondition2.length;

    binsMale.forEach(bin => bin.relativeFrequency = bin.length / totalMale);
    binsFemale.forEach(bin => bin.relativeFrequency = bin.length / totalFemale);
    binsMaleCondition2.forEach(bin => bin.relativeFrequency = bin.length / totalMaleCondition2);
    binsFemaleCondition2.forEach(bin => bin.relativeFrequency = bin.length / totalFemaleCondition2);


  // Y-Scale based on selectedScale
  const yDomain = selectedScale === "Frequency"
      ? [0, Math.max(
          d3.max(binsMale, d => d.length),
          d3.max(binsFemale, d => d.length),
          enableCondition2 ? d3.max(binsMaleCondition2, d => d.length) : 0,
          enableCondition2 ? d3.max(binsFemaleCondition2, d => d.length) : 0
        )]
      : [0, Math.max(
          d3.max(binsMale, d => d.relativeFrequency),
          d3.max(binsFemale, d => d.relativeFrequency),
          enableCondition2 ? d3.max(binsMaleCondition2, d => d.relativeFrequency) : 0,
          enableCondition2 ? d3.max(binsFemaleCondition2, d => d.relativeFrequency) : 0
        )];

  const y = d3.scaleLinear()
    .domain(yDomain)
    .range([height, margin.top]);

  const yTop = d3.scaleLinear()
    .domain(yDomain)
    .range([height / 2, 0]);

  const yBottom = d3.scaleLinear()
    .domain(yDomain)
    .range([height / 2, height]);

  const BAR_PADDING = 2;

  // Add patterns for dashed fill to the SVG
  svg.append("defs").html(`
    <pattern id="hashed-male" width="8" height="8" patternUnits="userSpaceOnUse">
      <path d="M0,0 L8,8" stroke="${MALE_COLOR_LIGHT}" stroke-width="2" />
      <path d="M8,0 L0,8" stroke="${MALE_COLOR_LIGHT}" stroke-width="2" />
    </pattern>
    <pattern id="hashed-female" width="8" height="8" patternUnits="userSpaceOnUse">
      <path d="M0,0 L8,8" stroke="${FEMALE_COLOR_LIGHT}" stroke-width="2" />
      <path d="M8,0 L0,8" stroke="${FEMALE_COLOR_LIGHT}" stroke-width="2" />
    </pattern>
  `);


  // Tooltip
  const tooltip = d3.select("body")
    .append("div")
    .attr("class", "tooltip")
    .style("position", "absolute")
    .style("opacity", 0)
    .style("background", "#333")
    .style("color", "#fff")
    .style("padding", "8px")
    .style("border-radius", "4px")
    .style("pointer-events", "none");

  // Add Bars for Male
  svg.selectAll(".bar-male")
    .data(binsMale)
    .enter().append("rect")
    .attr("class", "bar-male")
    .attr("x", d => x(d.x0) + BAR_PADDING / 2)
    .attr("y", d => selectedSide === "same"
      ? y(selectedScale === "Frequency" ? d.length : d.relativeFrequency)
      : yTop(selectedScale === "Frequency" ? d.length : d.relativeFrequency))
    .attr("width", d => Math.max(0, x(d.x1) - x(d.x0) - BAR_PADDING))
    .attr("height", d => selectedSide === "same"
      ? height - y(selectedScale === "Frequency" ? d.length : d.relativeFrequency)
      : yTop(0) - yTop(selectedScale === "Frequency" ? d.length : d.relativeFrequency))
    .attr("fill", MALE_COLOR)
    .on("mouseover", function (event, d) {
      d3.select(this)
        .attr("stroke", "black")
        .attr("stroke-width", 2)
        .attr("filter", "url(#glow)");
      tooltip.style("opacity", 1)
        .html(
          `Age Range: ${d.x0.toFixed(1)} - ${d.x1.toFixed(1)}<br>
          ${selectedScale === "Frequency" ? `Frequency: ${d.length}` : `Relative Frequency: ${(d.relativeFrequency * 100).toFixed(2)}%`}`
        )
        .style("left", `${event.pageX + 10}px`)
        .style("top", `${event.pageY - 20}px`);
    })
    .on("mousemove", function (event) {
      tooltip.style("left", `${event.pageX + 10}px`)
        .style("top", `${event.pageY - 20}px`);
    })
    .on("mouseout", function () {
      d3.select(this)
        .attr("stroke", "none")
        .attr("filter", "none");
      tooltip.style("opacity", 0);
    });

  // Add Bars for Female
  svg.selectAll(".bar-female")
    .data(binsFemale)
    .enter().append("rect")
    .attr("class", "bar-female")
    .attr("x", d => x(d.x0) + BAR_PADDING / 2)
    .attr("y", d => selectedSide === "same"
      ? y(selectedScale === "Frequency" ? d.length : d.relativeFrequency)
      : yBottom(0))
    .attr("width", d => Math.max(0, x(d.x1) - x(d.x0) - BAR_PADDING))
    .attr("height", d => selectedSide === "same"
      ? height - y(selectedScale === "Frequency" ? d.length : d.relativeFrequency)
      : yBottom(selectedScale === "Frequency" ? d.length : d.relativeFrequency) - yBottom(0))
    .attr("fill", FEMALE_COLOR)
    .attr("opacity", 0.7)
    .on("mouseover", function (event, d) {
      d3.select(this)
        .attr("stroke", "black")
        .attr("stroke-width", 2)
        .attr("filter", "url(#glow)");
      tooltip.style("opacity", 1)
        .html(
          `Age Range: ${d.x0.toFixed(1)} - ${d.x1.toFixed(1)}<br>
          ${selectedScale === "Frequency" ? `Frequency: ${d.length}` : `Relative Frequency: ${(d.relativeFrequency * 100).toFixed(2)}%`}`
        )
        .style("left", `${event.pageX + 10}px`)
        .style("top", `${event.pageY - 20}px`);
    })
    .on("mousemove", function (event) {
      tooltip.style("left", `${event.pageX + 10}px`)
        .style("top", `${event.pageY - 20}px`);
    })
    .on("mouseout", function () {
      d3.select(this)
        .attr("stroke", "none")
        .attr("filter", "none");
      tooltip.style("opacity", 0);
    });

    if (enableCondition2) {
      // Add Bars for Male
  svg.selectAll(".bar-male-condition2")
    .data(binsMaleCondition2)
    .enter().append("rect")
    .attr("class", "bar-male-condition2")
    .attr("x", d => x(d.x0) + BAR_PADDING / 2)
    .attr("y", d => selectedSide === "same"
      ? y(selectedScale === "Frequency" ? d.length : d.relativeFrequency)
      : yTop(selectedScale === "Frequency" ? d.length : d.relativeFrequency))
    .attr("width", d => Math.max(0, x(d.x1) - x(d.x0) - BAR_PADDING))
    .attr("height", d => selectedSide === "same"
      ? height - y(selectedScale === "Frequency" ? d.length : d.relativeFrequency)
      : yTop(0) - yTop(selectedScale === "Frequency" ? d.length : d.relativeFrequency))
    .attr("fill", "url(#hashed-male)")
    .attr("opacity", 0.5)
    .on("mouseover", function (event, d) {
      d3.select(this)
        .attr("stroke", "black")
        .attr("stroke-width", 2)
        .attr("filter", "url(#glow)");
      tooltip.style("opacity", 1)
        .html(
          `Age Range: ${d.x0.toFixed(1)} - ${d.x1.toFixed(1)}<br>
          ${selectedScale === "Frequency" ? `Frequency: ${d.length}` : `Relative Frequency: ${(d.relativeFrequency * 100).toFixed(2)}%`}`
        )
        .style("left", `${event.pageX + 10}px`)
        .style("top", `${event.pageY - 20}px`);
    })
    .on("mousemove", function (event) {
      tooltip.style("left", `${event.pageX + 10}px`)
        .style("top", `${event.pageY - 20}px`);
    })
    .on("mouseout", function () {
      d3.select(this)
        .attr("stroke", "none")
        .attr("filter", "none");
      tooltip.style("opacity", 0);
    });

  // Add Bars for Female
  svg.selectAll(".bar-female-condition2")
    .data(binsFemaleCondition2)
    .enter().append("rect")
    .attr("class", "bar-female-condition2")
    .attr("x", d => x(d.x0) + BAR_PADDING / 2)
    .attr("y", d => selectedSide === "same"
      ? y(selectedScale === "Frequency" ? d.length : d.relativeFrequency)
      : yBottom(0))
    .attr("width", d => Math.max(0, x(d.x1) - x(d.x0) - BAR_PADDING))
    .attr("height", d => selectedSide === "same"
      ? height - y(selectedScale === "Frequency" ? d.length : d.relativeFrequency)
      : yBottom(selectedScale === "Frequency" ? d.length : d.relativeFrequency) - yBottom(0))
    .attr("fill", "url(#hashed-female)")
    .attr("opacity", 0.7)
    .on("mouseover", function (event, d) {
      d3.select(this)
        .attr("stroke", "black")
        .attr("stroke-width", 2)
        .attr("filter", "url(#glow)");
      tooltip.style("opacity", 1)
        .html(
          `Age Range: ${d.x0.toFixed(1)} - ${d.x1.toFixed(1)}<br>
          ${selectedScale === "Frequency" ? `Frequency: ${d.length}` : `Relative Frequency: ${(d.relativeFrequency * 100).toFixed(2)}%`}`
        )
        .style("left", `${event.pageX + 10}px`)
        .style("top", `${event.pageY - 20}px`);
    })
    .on("mousemove", function (event) {
      tooltip.style("left", `${event.pageX + 10}px`)
        .style("top", `${event.pageY - 20}px`);
    })
    .on("mouseout", function () {
      d3.select(this)
        .attr("stroke", "none")
        .attr("filter", "none");
      tooltip.style("opacity", 0);
    });
    }

  // Add Axes
  const yLabel = (selectedScale === "Frequency") ? "Frequency" : "Relative Frequency (%)";

  if (selectedSide === "opposite") {
    // X-Axis
    const xAxis = svg.append("g")
      .attr("transform", `translate(0,${height / 2})`)
      .call(d3.axisBottom(x));
    xAxis.selectAll(".tick").filter(d => d === 0).remove();

    // Y-Axis for Male
    svg.append("g")
      .attr("transform", `translate(${margin.left},0)`)
      .call(d3.axisLeft(yTop))
      .append("text")
      .attr("x", -height / 4)
      .attr("y", -margin.left + 15)
      .attr("transform", "rotate(-90)")
      .attr("fill", "black")
      .style("font-size", "12px")
      .style("text-anchor", "middle")
      .text(`${yLabel} (Male)`);

    // Y-Axis for Female
    svg.append("g")
      .attr("transform", `translate(${margin.left},0)`)
      .call(d3.axisLeft(yBottom))
      .append("text")
      .attr("x", -height * 3/4)
      .attr("y", -margin.left + 15)
      .attr("transform", "rotate(-90)")
      .attr("fill", "black")
      .style("font-size", "12px")
      .style("text-anchor", "middle")
      .text(`${yLabel} (Female)`);
  } else {
    // X-Axis
    svg.append("g")
      .attr("transform", `translate(0,${height})`)
      .call(d3.axisBottom(x))
      .append("text")
      .attr("class", "x-axis-label")
      .attr("x", width / 2)
      .attr("y", margin.bottom - 10)
      .attr("fill", "black")
      .style("font-size", "16px")
      .style("text-anchor", "middle")
      .text("Age");

    // Single Y-Axis
    svg.append("g")
      .attr("transform", `translate(${margin.left},0)`)
      .call(d3.axisLeft(y))
      .append("text")
      .attr("class", "y-axis-label")
      .attr("x", -height / 2)
      .attr("y", -margin.left + 15)
      .attr("transform", "rotate(-90)")
      .attr("fill", "black")
      .style("font-size", "16px")
      .style("text-anchor", "middle")
      .text(yLabel);
  }
  // Add Legend
// Add Legend
const legend = svg.append("g").attr("class", "legend");

// Define legend items
const legendItems = [
  { label: "Male", color: MALE_COLOR, pattern: enableCondition2 ? "url(#hashed-male)" : null },
  { label: "Female", color: FEMALE_COLOR, pattern: enableCondition2 ? "url(#hashed-female)" : null },
];

const rectSize = 15; // Size of color rectangles
const itemSpacing = 20; // Spacing between items
const legendX = width - margin.right - 100; // Further right
const legendY = margin.top / 2; // Higher up

// Add legend items
legendItems.forEach((item, index) => {
  const group = legend
    .append("g")
    .attr("transform", `translate(${legendX}, ${legendY + index * itemSpacing})`);

  // Solid color rectangle
  group
    .append("rect")
    .attr("width", rectSize)
    .attr("height", rectSize)
    .attr("fill", item.color);

  // Hashed pattern rectangle (if applicable)
  if (item.pattern) {
    group
      .append("rect")
      .attr("x", rectSize + 5)
      .attr("width", rectSize)
      .attr("height", rectSize)
      .attr("fill", item.pattern);
  }

  // Label text
  group
    .append("text")
    .attr("x", item.pattern ? rectSize * 2 + 10 : rectSize + 10)
    .attr("y", rectSize / 2)
    .attr("dy", "0.35em")
    .attr("fill", "black")
    .style("font-size", "12px")
    .text(item.label);
});

}

  
</script>

<div class="visualization-container">
  <!-- Title for the visualization container -->
  <h2 class="visualization-title">
    {selectedScale === "Frequency" ? "Frequency" : "Relative Frequency"} of {selectedCondition1}
    {enableCondition2 ? ` and ${selectedCondition2}` : ""}
  </h2>
  <!-- SVG chart -->
  <svg id="visualization"></svg>
</div>

<style>
  .visualization-container {
    margin: 20px auto;
    padding: 20px;
    background-color: #f9f9f9;
    border: 1px solid #ddd;
    border-radius: 8px;
    max-width: 800px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    text-align: center; /* Center-align the title within the container */
  }

  .visualization-title {
    font-size: 18px;
    font-weight: bold;
    color: #333;
    margin-bottom: 15px; /* Add spacing between title and the SVG */
  }

  svg {
    width: 100%;
    height: auto;
  }

  .tooltip {
    font-size: 12px;
    font-family: Arial, sans-serif;
    text-align: left;
  }
</style>

