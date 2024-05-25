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

# 1.1. Analysis of mastery level of knowledge points

```js
const problems = await FileAttachment("data/problems.csv").csv({typed: true});

const students = await FileAttachment("data/students.csv").csv({typed: true});
```

```js
const search = view(Inputs.search(students, {label: "Fuzzy search for student id", placeholder: "Search students...", columns: ["student_ID"]}));
```

```js
const studentID = view(Inputs.select(search.map(d => d.student_ID), {label: "Select Student"}));
```

```js
const answerData = await FileAttachment("data/answer1.csv").csv({typed: true});
```

```js
const studentAnswerDataRow = answerData.filter(d => d.student_ID === studentID);

const specificStatus = "Absolutely_Correct";
const getScore = "score_submit";
const proScore = "score_problem";
const statusFields = [
  "Absolutely_Correct",
  "Absolutely_Error",
  "Error1", "Error2", "Error3", "Error4", "Error5",
  "Error6", "Error7", "Error8", "Error9", "Partially_Correct"
];

// 遍历学生的所有答题记录
studentAnswerDataRow.forEach(row => {
  // 计算所有状态的返回次数总和
  const totalSubmissions = statusFields.reduce((total, field) => {
    return total + row[field];
  }, 0);

  // 特定状态的返回次数
  const specificStatusCount = row[specificStatus];

  // 避免除以零
  const ratio = totalSubmissions > 0 ? (specificStatusCount / totalSubmissions) : 0;

  // 添加新字段存储比值
  row[`${specificStatus}_ratio`] = ratio;

  row[`score_ratio`] = row[getScore] / row[proScore] / totalSubmissions;
});
```

```js
const buildRenderAnswerDataDiagram = (config) => {
  const width = config.width;
  const height = config.height;
  const marginTop = config.margin.top;
  const marginRight = config.margin.right;
  const marginBottom = config.margin.bottom;
  const marginLeft = config.margin.left;

  // Create the SVG container.
  const svg = d3.create("svg")
      .attr("viewBox", [0, 0, width, height])
      .attr("style", "max-width: 100%; height: auto; font: 10px sans-serif;");


  return (data) => {
    // Prepare the scales for positional encoding.
    const x = d3.scaleLinear()
        .domain(d3.extent(studentAnswerDataRow, d => d.score_ratio)).nice()
        .range([marginLeft, width - marginRight]);

    const y = d3.scaleLinear()
        .domain(d3.extent(studentAnswerDataRow, d => d.Absolutely_Correct_ratio)).nice()
        .range([height - marginBottom, marginTop]);

    // Create the axes.
    svg.append("g")
    .attr("transform", `translate(0,${height - marginBottom})`)
    .call(d3.axisBottom(x).ticks(width / 80))
    .call(g => g.select(".domain").remove())
    .call(g => g.append("text")
        .attr("x", width)
        .attr("y", marginBottom - 4)
        .attr("fill", "currentColor")
        .attr("text-anchor", "end")
        .text("Answering status →"));

    svg.append("g")
    .attr("transform", `translate(${marginLeft},0)`)
    .call(d3.axisLeft(y))
    .call(g => g.select(".domain").remove())
    .call(g => g.append("text")
        .attr("x", -marginLeft)
        .attr("y", 10)
        .attr("fill", "currentColor")
        .attr("text-anchor", "start")
        .text("↑ Score_Ratio"));

    // Create the grid.
    svg.append("g")
    .attr("stroke", "currentColor")
    .attr("stroke-opacity", 0.1)
    .call(g => g.append("g")
        .selectAll("line")
        .data(x.ticks())
        .join("line")
        .attr("x1", d => 0.5 + x(d))
        .attr("x2", d => 0.5 + x(d))
        .attr("y1", marginTop)
        .attr("y2", height - marginBottom))
    .call(g => g.append("g")
        .selectAll("line")
        .data(y.ticks())
        .join("line")
        .attr("y1", d => 0.5 + y(d))
        .attr("y2", d => 0.5 + y(d))
        .attr("x1", marginLeft)
        .attr("x2", width - marginRight));

    // Create the grid.
    svg.append("g")
        .attr("stroke", "currentColor")
        .attr("stroke-opacity", 0.1)
        .call(g => g.append("g")
        .selectAll("line")
        .data(x.ticks())
        .join("line")
            .attr("x1", d => 0.5 + x(d))
            .attr("x2", d => 0.5 + x(d))
            .attr("y1", marginTop)
            .attr("y2", height - marginBottom))
        .call(g => g.append("g")
        .selectAll("line")
        .data(y.ticks())
        .join("line")
            .attr("y1", d => 0.5 + y(d))
            .attr("y2", d => 0.5 + y(d))
            .attr("x1", marginLeft)
            .attr("x2", width - marginRight));

    // Add a layer of dots.
    svg.append("g")
        .attr("stroke", "steelblue")
        .attr("stroke-width", 1.5)
        .attr("fill", "blue")
        .selectAll("circle")
        .data(studentAnswerDataRow)
        .join("circle")
        .attr("cx", d => x(d.Absolutely_Correct_ratio))
        .attr("cy", d => y(d.score_ratio))
        .attr("r", 3);

    // Add a layer of labels.
    // svg.append("g")
    //     .attr("font-family", "sans-serif")
    //     .attr("font-size", 10)
    //     .selectAll("text")
    //     .data(studentAnswerDataRow)
    //     .join("text")
    //     .attr("dy", "0.35em")
    //     .attr("x", d => x(d.Absolutely_Correct_ratio) + 7)
    //     .attr("y", d => y(d.score_ratio))
    //     .text(d => d.title_ID);
        
    console.log(studentAnswerDataRow);

    return svg.node();

    

    // const handleOnMouseEnter = (e, d) => {
    //   // make other area opacity
    //   chart.selectAll(".bar").filter(d2 => d2.hour !== d.hour).attr("opacity", "0.3");
    //   // show tooltip
    //   d3.select("#tooltip").style("display", "block");
    // }

    // const handleOnMouseMove = (e, d) => {
    //   console.log(d);

    //   d3.select("#tooltip").style("left", e.pageX + 10 + "px").style("top", e.pageY + 10 + "px").html(`<div class="tooltip-label">Submit Times</div>${d.submits}`);
    // }

    // const handleOnMouseLeave = (e, d) => {
    //   // make other area opaque
    //   chart.selectAll(".bar").attr("opacity", "1");
    //   // hide tooltip
    //   d3.select("#tooltip").style("display", "none");
    // }

    // const barChart = chart.selectAll(".bar").data(data).join("rect").attr("class", "bar").attr("fill", "steelblue").attr("width", xScale.bandwidth()).attr("x", d => xScale(d.hour));
    // barChart.transition().duration(500).attr("height", d => height - yScale(d.submits)).attr("y", d =>  yScale(d.submits));
    // barChart.on("mouseenter", handleOnMouseEnter).on("mousemove", handleOnMouseMove).on("mouseleave", handleOnMouseLeave)

    // return svg.node();
  }


}
```

```js
const renderAnswerDataDiagram = buildRenderAnswerDataDiagram({margin: {left: 55, right: 20, top: 20, bottom: 55}, width: 1000, height: 600});
```

```js
display(renderAnswerDataDiagram(studentAnswerDataRow));
```