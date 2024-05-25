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

# 2.3. Problem Accuracy Analysis


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
const problemAccuracies = await FileAttachment("data/problem_accuracies.csv").csv({typed: true});
```

```js
const studentProblemAccuraciesRow = problemAccuracies.filter(d => d.student_ID === studentID)[0];

```
```js
const accuracySubmit = [
  {
    name: "Correct",
    value: studentProblemAccuraciesRow.accuracy_submit
  },
  {
    name: "Incorrect",
    value: 1 - studentProblemAccuraciesRow.accuracy_submit
  }
];
```
```js
const accuracyProblem = [
  {
    name: "Correct",
    value: studentProblemAccuraciesRow.accuracy_problem
  },
  {
    name: "Incorrect",
    value: 1 - studentProblemAccuraciesRow.accuracy_problem
  }
]
```

```js
const buildRenderPieChart = (config) => {
  const width = config.width;
  const height = config.height;

  // Create the color scale.
  const colorScale = d3.scaleOrdinal()
      .range(["lightblue", "lightgreen"])

  // Create the pie layout and arc generator.
  const pie = d3.pie()
      .sort(null)
      .value(d => d.value);

  const arc = d3.arc()
      .innerRadius(0)
      .outerRadius(Math.min(width, height) / 2 - 1);

  const labelRadius = arc.outerRadius()() * 0.8;

  // A separate arc generator for labels.
  const arcLabel = d3.arc()
      .innerRadius(labelRadius)
      .outerRadius(labelRadius);

  // Create the SVG container.
  const svg = d3.create("svg")
      .attr("width", width)
      .attr("height", height)
      .attr("viewBox", [-width / 2, -height / 2, width, height])
      .attr("style", "max-width: 100%; height: auto; font: 25px sans-serif;");

  function arcTween(a) {
    const i = d3.interpolate(this._current, a);
    this._current = i(0);
    return (t) => arc(i(t));
  }

  return (data) => {
    colorScale.domain(data.map(d => d.name))

    const arcs = pie(data);

    const handleOnMouseEnter = (e, d) => {
      // make other area opacity
      svg.selectAll(".pie").filter(d2 => d2.data.name !== d.data.name).attr("opacity", "0.3");
      // show tooltip
      d3.select("#tooltip").style("display", "block");
    }

    const handleOnMouseMove = (e, d) => {
      d3.select("#tooltip").style("left", e.pageX + 10 + "px").style("top", e.pageY + 10 + "px").html(`<div class="tooltip-label">Type</div>${d.data.name}<div class="tooltip-label">Percentage</div>${(d.data.value * 100).toLocaleString("en-US")}%`);
    }

    const handleOnMouseLeave = (e, d) => {
      // make other area opaque
      svg.selectAll(".pie").attr("opacity", "1");
      // hide tooltip
      d3.select("#tooltip").style("display", "none");
    }

    // Add a sector path for each value.
    const pieChart = svg.selectAll(".pie").data(arcs).join("path").attr("class", "pie").attr("fill", d => colorScale(d.data.name)).attr("d", arc);
    pieChart.transition().duration(500).attrTween("d", arcTween)
    pieChart.on("mouseenter", handleOnMouseEnter).on("mousemove", handleOnMouseMove).on("mouseleave", handleOnMouseLeave)

    // Create a new arc generator to place a label close to the edge.
    // The label shows the value if there is enough room.
    const pieText = svg.selectAll(".pie-text").data(arcs).join("text").attr("class", "pie-text").attr("text-anchor", "middle").text(d => d.data.value > 0 ? d.data.name : "");
    pieText.transition().duration(500).attr("transform", d => `translate(${arcLabel.centroid(d)})`);
    
    return svg.node();
  }
}
```

```js
const renderPieChart1 = buildRenderPieChart({margin: {left: 55, right: 20, top: 20, bottom: 55}, width: 1000, height: 600});;
```


```js
const renderPieChart2 = buildRenderPieChart({margin: {left: 55, right: 20, top: 20, bottom: 55}, width: 1000, height: 600});
```

<div class="grid grid-cols-2">
  <div class="card"><h4>Accuracy of Submission</h4>${renderPieChart1(accuracySubmit)}</div>
  <div class="card"><h4>Accuracy of Problem</h4>${renderPieChart2(accuracyProblem)}</div>
</div>
