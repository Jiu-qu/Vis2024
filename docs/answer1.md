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
const err = "Absolutely_Error";
const err1 = "Error1";
const err2 = "Error2";
const err3 = "Error3";
const err4 = "Error4";
const err5 = "Error5";
const err6 = "Error6";
const err7 = "Error7";
const err8 = "Error8";
const err9 = "Error9";
const partCorrect = "Partially_Correct";
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
  const status_num = row[specificStatus] * 0.0 + row[err] * 0.4 + row[partCorrect] * 0.20 + row[err1] * 0.17 + row[err2] * 0.165 + row[err3] * 0.160 + row[err4] * 0.155 + row[err5] * 0.150 + row[err6] * 0.145 + row[err7] * 0.140 + row[err8] * 0.135 + row[err9] * 0.130;

  console.log(status_num);

  // 避免除以零
  const ratio = totalSubmissions > 0 ? (specificStatusCount / totalSubmissions) : 0;

  // 添加新字段存储比值
  row[`${specificStatus}_ratio`] = status_num;
  row[`total_time`] = totalSubmissions;

  row[`score_ratio`] = row[getScore] / row[proScore];
});
```

```js
const collisionData = {};
function collectCollisionData(data, xScale, yScale) {
  data.forEach(d => {
    const key = `${xScale(d.Absolutely_Correct_ratio)},${yScale(d.score_ratio)}`;
    if (!collisionData[key]) {
      collisionData[key] = [];
    }
    collisionData[key].push(d); // 收集重合的数据点
  });
}
let hoveredData;
const collisionCounts = studentAnswerDataRow.reduce((acc, d) => {
  const key = `${d.Absolutely_Correct_ratio}_${d.score_ratio}`;
  acc[key] = (acc[key] || 0) + 1;
  return acc;
}, {});
// const colorScale = d3.scaleQuantize()
//     .domain([1, d3.max(Object.values(collisionCounts))]) // 设置颜色比例尺的域
//     .range(["#ffeda0", "#fdae61", "#f46d43", "#d73027"]); 
// const colorScale = d3.scaleSequential()
//   .interpolator(d3.interpolateRainbow) // 选择一个彩色的插值方案
//   .domain([1, d3.max(Object.values(collisionCounts))]) // 定义域为重合数量的最小和最大值

const colors = d3.schemeTableau10;

// 2. 创建 scaleOrdinal 颜色比例尺
const knowledgeColorScale = d3.scaleOrdinal()
  .domain(["t5V9e", "m3D1v", "g7R2j", "y9W5d", "r8S3g", "b3C9s", "k4W1c", "s8Y2f"]) // 知识字段的八个值
  .range(colors); // 为每个知识值分配颜色

const radiusScale = (c) => {
  return 8 * Math.sqrt(c);
}

d3.scaleSqrt()
  .domain([1, d3.max(Object.values(collisionCounts))])
  .range([5, 20]); // 将重合数量映射到半径大小的范围，例如从 5px 到 20px

studentAnswerDataRow.sort((a, b) =>
  collisionCounts[`${b.Absolutely_Correct_ratio}_${b.score_ratio}`] -
  collisionCounts[`${a.Absolutely_Correct_ratio}_${a.score_ratio}`]
);

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
        .domain(d3.extent(studentAnswerDataRow, d => d.Absolutely_Correct_ratio)).nice()
        .range([marginLeft, width - marginRight]);

    const y = d3.scaleLinear()
        .domain([0, 1]).nice()
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

    const points = svg.append("g")
        .selectAll("circle")
        .data(studentAnswerDataRow)
        .join("circle")
        .attr("cx", d => x(d.Absolutely_Correct_ratio))
      .attr("cy", d => y(d.score_ratio))
      .attr("r", 0) // 开始时半径为0
      .attr("fill", d => knowledgeColorScale(d.knowledge))
      .attr("opacity", 0.8)

    points.transition() // 开始过渡效果
      .duration(500) // 设置动画持续时间
      .attr("cx", d => x(d.Absolutely_Correct_ratio))
      .attr("cy", d => y(d.score_ratio))
      .attr("r", d => radiusScale(collisionCounts[`${d.Absolutely_Correct_ratio}_${d.score_ratio}`]))
      .attr("fill", d => knowledgeColorScale(d.knowledge));

    // Add a layer of dots.
    // svg.append("g")
    //     .attr("stroke", "steelblue")
    //     .attr("stroke-width", 1.5)
    //     .selectAll("circle")
    //     .data(studentAnswerDataRow)
    //     .join("circle")
    //     .attr("cx", d => x(d.Absolutely_Correct_ratio))
    //     .attr("cy", d => y(d.score_ratio))
    //     .attr("r", d => radiusScale(collisionCounts[`${d.Absolutely_Correct_ratio}_${d.score_ratio}`]))
    //     .attr("fill", d => knowledgeColorScale(d.knowledge))
    //     .style("z-index", d => -radiusScale(collisionCounts[`${d.Absolutely_Correct_ratio}_${d.score_ratio}`]))
        // .attr("fill", d => {
        //   const key = `${d.Absolutely_Correct_ratio}_${d.score_ratio}`;
        //   return colorScale(collisionCounts[key]);
        // });

    collectCollisionData(studentAnswerDataRow, x, y);

    svg.selectAll("circle")
        .on("mouseover", (event, d) => {
          hoveredData = d;

          const positionKey = `${x(d.Absolutely_Correct_ratio)},${y(d.score_ratio)}`;
          const collidingData = collisionData[positionKey] || [d]; // 获取重合的数据点
          const content = collidingData.map(dataPoint => `
            <h3>Question: ${dataPoint.title_ID}</h3>
            Score Ratio: ${dataPoint.score_ratio}
            <br>Submit Times: ${dataPoint.total_time}
            <br>
            Knowledge: ${dataPoint.sub_knowledge}
            <br>
          `).join('');
          tooltip.style("display", "block")
                .html(content);
          
          // 降低其他点的透明度
          svg.selectAll("circle").filter(p => p !== d).attr("opacity", 0.3);

        })
        .on("mouseout", (event, d) => {
          tooltip.style("display", "none");
          svg.selectAll("circle").attr("opacity", 0.8);
        })
        .on("mousemove", (event, d) => {
          // 更新Tooltip内容和位置
          tooltip.style("left", `${event.pageX + 10}px`)
                  .style("top", `${event.pageY + 10}px`);
        });

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
        
    // console.log(studentAnswerDataRow);

    return svg.node();
  }


}
```

```js
const renderAnswerDataDiagram = buildRenderAnswerDataDiagram({margin: {left: 55, right: 20, top: 20, bottom: 55}, width: 1000, height: 600});
```

```js
display(renderAnswerDataDiagram(studentAnswerDataRow));
```