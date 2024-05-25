---
title: Example report
---

# A brief history of space exploration

This report is a brief overview of the history and current state of rocket launches and space exploration.

## Background

The history of rocket launches dates back to ancient China, where gunpowder-filled tubes were used as primitive forms of propulsion.

Fast-forward to the 20th century during the Cold War era, the United States and the Soviet Union embarked on a space race, a competition to innovate and explore beyond Earth.

This led to the launch of the first artificial satellite, Sputnik 1, and the crewed moon landing by Apollo 11. As technology advanced, rocket launches became synonymous with space exploration and satellite deployment.

## The Space Shuttle era

```js
import {timeline} from "./components/timeline.js";
```

```js
const events = FileAttachment("./data/events.json").json();
```

```js
timeline(events, {height: 300})
```

### Sputnik 1 (1957)

This was the first artificial satellite. Launched by the Soviet Union, it marked the beginning of the space age.

### Apollo 11 (1969)

The historic Apollo 11 mission, led by NASA, marked the first successful human landing on the Moon. Astronauts Neil Armstrong and Buzz Aldrin became the first humans to set foot on the lunar surface.

### Viking 1 and 2 (1975)

NASA’s Viking program successfully launched two spacecraft, Viking 1 and Viking 2, to Mars. These missions were the first to successfully land and operate on the Martian surface, conducting experiments to search for signs of life.

### Space Shuttle Columbia (1981)

The first orbital space shuttle mission, STS-1, launched the Space Shuttle Columbia on April 12, 1981. The shuttle program revolutionized space travel, providing a reusable spacecraft for a variety of missions.

### Hubble Space Telescope (1990)

The Hubble Space Telescope has provided unparalleled images and data, revolutionizing our understanding of the universe and contributing to countless astronomical discoveries.

### International Space Station (ISS) construction (1998—2011)

The ISS, a collaborative effort involving multiple space agencies, began construction with the launch of its first module, Zarya, in 1998. Over the following years, various modules were added, making the ISS a symbol of international cooperation in space exploration.

## Commercial spaceflight

After the Space Shuttle program, a new era emerged with a shift towards commercial spaceflight.

Private companies like Blue Origin, founded by Jeff Bezos in 2000, and SpaceX, founded by Elon Musk in 2002, entered the scene. These companies focused on developing reusable rocket technologies, significantly reducing launch costs.

SpaceX, in particular, achieved milestones like the first privately developed spacecraft to reach orbit (Dragon in 2010) and the first privately funded spacecraft to dock with the ISS (Dragon in 2012).

## Recent launch activity

The proliferation of commercial space companies has driven a surge in global launch activity within the last few years.

SpaceX’s Falcon 9 and Falcon Heavy, along with other vehicles from companies like Rocket Lab, have become workhorses for deploying satellites, conducting scientific missions, and ferrying crew to the ISS.

The advent of small satellite constellations, such as Starlink by SpaceX, has further fueled this increase in launches. The push for lunar exploration has added momentum to launch activities, with initiatives like NASA’s Artemis program and plans for crewed missions to the Moon and Mars.

## Looking forward

As technology continues to advance and global interest in space exploration grows, the future promises even more exciting developments in the realm of rocket launches and space travel.

Exploration will not only be limited to the Moon or Mars, but extend to other parts of our solar system such as Jupiter and Saturn’s moons, and beyond.

<svg class='sand'> </svg> 

```js
// Specify the chart’s dimensions.
const width = 928;
const height = 600;
const marginTop = 25;
const marginRight = 20;
const marginBottom = 35;
const marginLeft = 40;

const cars = FileAttachment("mtcars.csv").csv({typed: true})

console.log(cars)

// Prepare the scales for positional encoding.
const x = d3.scaleLinear()
.domain([0,100]).nice()
.range([marginLeft, width - marginRight]);

const y = d3.scaleLinear()
.domain([0,100]).nice()
.range([height - marginBottom, marginTop]);


// Create the SVG container.
const svg = d3.select(".sand")
    .attr("viewBox", [0, 0, width, height])
    .attr("style", "max-width: 100%; height: auto; font: 10px sans-serif;");

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
    .text("↑ Score"));

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
    .attr("fill", "none")
.selectAll("circle")
.data(cars)
.join("circle")
    .attr("cx", d => x(d.status))
    .attr("cy", d => y(d.score))
    .attr("r", 3);

// Add a layer of labels.
svg.append("g")
    .attr("font-family", "sans-serif")
    .attr("font-size", 10)
.selectAll("text")
.data(cars)
.join("text")
    .attr("dy", "0.35em")
    .attr("x", d => x(d.status) + 7)
    .attr("y", d => y(d.score))
    .text(d => d.name);
```

