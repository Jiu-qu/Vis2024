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

# 2.2. Preferred Question Type Analysis


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
const questionTypes = await FileAttachment("data/question_types.csv").csv({typed: true});
```

```js
const dictToArray = (data) => {
    return Object.getOwnPropertyNames(data).filter(d => d !== "student_ID").map(d => {return {knowledge: d, freq: data[d]}});
}

const groupKnowledges = (data) => {
    return d3.groups(dictToArray(data), d => d.knowledge.split("_")[0]);
}

const formatKnowledges = (data) => {
    return groupKnowledges(data).map(d => d[1].map((d2, no) => {return {knowledge: d[0], name: d2.knowledge, pos: no, freq: d2.freq}})).flat()
}
```

```js
const studentQuestionTypesRow = questionTypes.filter(d => d.student_ID === studentID)[0];
```

```js
const studentQuestionTypes = formatKnowledges(studentQuestionTypesRow);
```

```js
const buildRenderQuestionTypesDiagram = (config) => {
  const margin = config.margin;
  const width = config.width - margin.left - margin.right;
  const height = config.height - margin.top - margin.bottom;

  const svg = d3.create("svg").attr("width", config.width).attr("height", config.height);

  const chart = svg.append("g").attr("transform", `translate(${margin.left}, ${margin.top})`);

  const yAxisGroup = chart.append("g").attr("transform", "translate(-2, 0)");

  // scale
  const xScale = d3.scaleBand().range([0, width]).paddingInner(0.05);
  const yScale = d3.scaleBand().range([height, 0]).paddingInner(0.05);
  const colorScale = d3.scaleSequential().interpolator(d3.interpolateBuGn);

  const yAxis = d3.axisLeft(yScale).tickSizeOuter(0);

  return (data) => {
    xScale.domain(data.map(d => d.pos));
    yScale.domain(data.map(d => d.knowledge));
    colorScale.domain([0.7, 1.2]);

    yAxisGroup.call(yAxis);
    yAxisGroup.select('.domain').attr('stroke-width', 0);


    const handleOnMouseEnter = (e, d) => {
      // make other area opacity
      chart.selectAll(".bar").filter(d2 => d2.hour !== d.hour).attr("opacity", "0.3");
      // show tooltip
      d3.select("#tooltip").style("display", "block");
    }

    const handleOnMouseMove = (e, d) => {
      console.log(d);

      d3.select("#tooltip").style("left", e.pageX + 10 + "px").style("top", e.pageY + 10 + "px").html(`<div class="tooltip-label">Knowledge</div>${d.name}<div class="tooltip-label">Frequency</div>${d.freq.toFixed(2)}`);
    }

    const handleOnMouseLeave = (e, d) => {
      // make other area opaque
      chart.selectAll(".bar").attr("opacity", "1");
      // hide tooltip
      d3.select("#tooltip").style("display", "none");
    }

    const cell = chart.selectAll(".h-cell").data(data).join("rect").attr("class", d => d.pos === 0 ? "h-cell" : "h-cell h-cell-sub").attr("height", yScale.bandwidth()).attr("width", xScale.bandwidth()).attr("x", d => xScale(d.pos)).attr("y", d => yScale(d.knowledge)).attr("fill", d => colorScale(d.freq));
    cell.on("mouseenter", handleOnMouseEnter).on("mousemove", handleOnMouseMove).on("mouseleave", handleOnMouseLeave);

    const cellText = chart.selectAll(".h-text").data(data).join("text").attr("class", "h-text").attr("x", d => xScale(d.pos) + xScale.bandwidth() / 2).attr("y", d => yScale(d.knowledge) + yScale.bandwidth() / 2).attr("fill", d => d.freq > 0.8 ? "white" : "black").attr("text-anchor", "middle").text(d => d.pos === 0 ? d.name : d.name.split("_")[1])

    return svg.node();
  }
}
```

```js
const renderQuestionTypesDiagram = buildRenderQuestionTypesDiagram({margin: {left: 60, right: 20, top: 20, bottom: 20}, width: 400, height: 600});
```

```js
display(renderQuestionTypesDiagram(studentQuestionTypes));
```