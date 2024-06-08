---
toc: false
---

<style>
#tooltip {
  position: absolute;
  display: none;
  background: #fff;
  box-shadow: 2px 2px 3px 0px rgb(92 92 92 / 0.5);
  border: 1px solid #ddd;
  font-size: 12px;
  font-weight: 600;
  padding: 2px 8px;
}

.tooltip-label {
  font-weight: 500;
  font-size: 10px;
  color: #888;
}
</style>

```js
const tooltip = d3.select("body").append("div").attr("id", "tooltip");
```

# 2.1. Submission Frequency Analysis

```js
const students = await FileAttachment("data/students.csv").csv({typed: true});
```

```js
const search = view(Inputs.search(students, {label: "Fuzzy search for student id", placeholder: "Search students...", columns: ["student_ID"]}));
```

```js
const studentID = view(Inputs.select(search.map(d => d.student_ID), {label: "Select Student"}));
```

```js
const submitTimes = await FileAttachment("data/submit_times.csv").csv({typed: true});
```

```js
const studentSubmitTimesRow = submitTimes.filter(d => d.student_ID === studentID)[0];

const studentSubmitTimes = [...Array(24).keys()].map(d => {return {hour: d, submits: studentSubmitTimesRow[`${d}h`]}});
```

```js
const buildRenderSubmitTimesDiagram = (config) => {
  const margin = config.margin;
  const width = config.width - margin.left - margin.right;
  const height = config.height - margin.top - margin.bottom;

  const svg = d3.create("svg").attr("width", config.width).attr("height", config.height);

  const chart = svg.append("g").attr("transform", `translate(${margin.left}, ${margin.top})`);

  // axis group
  const xAxisGroup = chart.append("g").attr("transform", `translate(0, ${height})`);
  const yAxisGroup = chart.append("g");

  // axis label
  const labelEmbed = 40;

  const xAxisLabel = xAxisGroup.append("text").attr("class", "axis-label").attr("x", width / 2).attr("y", labelEmbed).attr("fill", "black").text("Hours");
  const yAxisLabel = yAxisGroup.append("text").attr("class", "axis-label").attr("transform", "rotate(-90)").attr("x", -height / 2 + labelEmbed).attr("y", -labelEmbed).attr("fill", "black").text("Submit Times");

  // scale
  const xScale = d3.scaleBand().range([0, width]).paddingInner(0.05);
  const yScale = d3.scaleLinear().range([height, 0]);
  const lineXScale = d3.scaleLinear().range([0, width]);

  // axis
  const xAxis = d3.axisBottom(xScale).tickSizeOuter(0).tickFormat(d => d + "h");
  const yAxis = d3.axisLeft(yScale).tickSizeOuter(0);

  return (data) => {
    xScale.domain(data.map(d => d.hour));
    yScale.domain([0, d3.max(data, d => d.submits)]);
    lineXScale.domain([0, d3.max(data, d => d.hour)]);

    yAxisGroup.transition().duration(500).call(yAxis);
    xAxisGroup.call(xAxis);

    const handleOnMouseEnter = (e, d) => {
      // make other area opacity
      chart.selectAll(".bar").filter(d2 => d2.hour !== d.hour).attr("opacity", "0.3");
      // show tooltip
      d3.select("#tooltip").style("display", "block");
    }

    const handleOnMouseMove = (e, d) => {
      console.log(d);

      d3.select("#tooltip").style("left", e.pageX + 10 + "px").style("top", e.pageY + 10 + "px").html(`<div class="tooltip-label">Submit Times</div>${d.submits}`);
    }

    const handleOnMouseLeave = (e, d) => {
      // make other area opaque
      chart.selectAll(".bar").attr("opacity", "1");
      // hide tooltip
      d3.select("#tooltip").style("display", "none");
    }

    const barChart = chart.selectAll(".bar").data(data).join("rect").attr("class", "bar").attr("fill", "#8dd3c7").attr("width", xScale.bandwidth()).attr("x", d => xScale(d.hour));
    barChart.transition().duration(500).attr("height", d => height - yScale(d.submits)).attr("y", d =>  yScale(d.submits));
    barChart.on("mouseenter", handleOnMouseEnter).on("mousemove", handleOnMouseMove).on("mouseleave", handleOnMouseLeave)

    const middleValue = data.reduce((acc, d) => acc + d.hour * d.submits, 0) / data.reduce((acc, d) => acc + d.submits, 0);

    const middleValueLine = chart.selectAll(".middle-val-line").data([middleValue]).join("line").attr("class", "middle-val-line").attr("y1", 0).attr("y2", height).style("stroke-width", 2).style("stroke", "#8da0cb").style("fill", "none");
    middleValueLine.transition().duration(500).attr("x1", d => lineXScale(d)).attr("x2", d => lineXScale(d))

    const middleValueText = chart.selectAll(".middle-val-text").data([middleValue]).join("text").attr("class", "middle-val-text").attr("text-anchor", "middle").attr("y", -20).attr("dy", "0.85em").text("Average Submit Time").attr("fill", "#8da0cb");
    middleValueText.transition().duration(500).attr("x", d => lineXScale(d));

    return svg.node();
  }
}
```

```js
const renderSubmitTimesDiagram = buildRenderSubmitTimesDiagram({margin: {left: 55, right: 20, top: 20, bottom: 55}, width: 1000, height: 600});
```

```js
display(renderSubmitTimesDiagram(studentSubmitTimes));
```