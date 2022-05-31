The data is drawn in three different ways simultaneously:

- as a **curve**, which presents a nuanced histogram
- as a **box plot**, marking the 10th, 25th, 50th (aka median), 75th, and 90th percentiles
- as **raw values**, allowing users to interact with individual elements and potentially revealing subtle patterns which aren't visible at the top level from the other views

This project is written in a heavily annotated style called [literate programming](https://en.wikipedia.org/wiki/Literate_programming). The code blocks from this Markdown document are being executed as JavaScript by [lit-web](https://github.com/vijithassar/lit-web).

## Implementation

First, an anonymous function to guard the rest of the code in this script within a closure.

```javascript
(() => {
```

### Configuration

Configuration variables, mostly used for positioning.

```javascript
    const height = 450
    const total_width = window.innerWidth
    const grid = 10
    const margin = {
    top: grid * 4,
    right: grid,
    bottom: grid,
    left: grid
    }
    const width = total_width - margin.left - margin.right
    const segment = height * 0.25
    const size = segment * 0.5
```

### Wrapper

Each raincloud plot consists of three distinct sections, and each section can be encapsulated in a separate function for the sake of clarity. Wrapping these into a single parent function makes it easier to render the whole thing in one shot, and it means the data only needs to be provided as a parameter once.

```javascript
  const raincloud = (selection, data) => {
    selection
      .call(curve, data)
      .call(dots, data)
      .call(boxplot, data)
  }
```
### Data
Data is given to dataA, dataB and dataC.

### Curve

To plot the **curve** section, first group values into a histogram and then bind it to a path drawn with `d3.area()` or `d3.line()`.

```javascript
  const curve = (selection, data) => {
    const histogram = d3.histogram()
      .thresholds(20)
      (data)
      .map(bin => bin.length)
    const x = d3.scaleLinear()
      .domain([0, histogram.length])
      .range([0, width])
    const y = d3.scaleLinear()
      .domain([0, d3.max(histogram)])
      .range([size, 0])
    const area = d3.area()
      .y0(y)
      .y1(size)
      .x((d, i) => x(i))
      .curve(d3.curveBasis)
    selection.append('g')
      .classed('curve', true)
      .datum(histogram)
      .append('path')
      .attr('d', area)
  }
```

### Box Plot

The **box plot** section is graphically simple but can be tedious to draw because you need to carefully juggle a bunch of line segments. For this step `d3.quantile` does most of the heavy lifting.

```javascript
  const boxplot = (selection, datareq) => {
    const data = datareq
      .sort(d3.ascending) //for data sort
    const bar = grid
    const x = d3.scaleLinear()
      .domain(d3.extent(data))
      .range([0, width])
    const plot = selection
      .append('g')
      .classed('boxplot', true)
      .attr('transform', `translate(0,${segment * 0.75 - grid})`)
    plot
      .append('line')
      .attr('x1', x(d3.quantile(data, 0.5)))
      .attr('x2', x(d3.quantile(data, 0.5)))
      .attr('y1', 0)
      .attr('y2', bar)
    plot
      .append('line')
      .attr('x1', x(d3.quantile(data, 0.1)))
      .attr('x2', x(d3.quantile(data, 0.25)))
      .attr('y1', bar * 0.5)
      .attr('y2', bar * 0.5)
    plot
      .append('line')
      .attr('x1', x(d3.quantile(data, 0.9)))
      .attr('x2', x(d3.quantile(data, 0.75)))
      .attr('y1', bar * 0.5)
      .attr('y2', bar * 0.5)
    plot.append('rect')
      .attr('x', x(d3.quantile(data, 0.25)))
      .attr('y', 0)
      .attr('height', bar)
      .attr('width', x(d3.quantile(data, 0.75)) - x(d3.quantile(data, 0.25)))
  }
```

### Raw Data

The **raw data** section plots the data according to the primary quantitative axis; the second value can be randomized, or else the points can be stacked ordinally. The DOM API can be a performance bottleneck if there's a very large number of data points, in which case it may be worthwhile to try rendering this part using the [`canvas` element](https://en.wikipedia.org/wiki/Canvas_element).

```javascript
  const dots = (selection, data) => {
    const x = d3.scaleLinear()
      .domain(d3.extent(data))
      .range([0, width])
    selection
      .append('g')
      .classed('dots', true)
      .attr('transform', `translate(0,${segment * 0.5 + grid})`)
      .selectAll('circle')
      .data(data)
      .enter()
      .append('circle')
      .attr('r', 2)
      .attr('cx', x)
      .attr('cy', () => Math.random() * size * 0.5)
  }
```

### DOM

A small helper function to add the SVG and complete the initial DOM setup.

```javascript
  const dom = selection => {
    selection
      .append('svg')
      .attr('height', height)
      .attr('width', width)
  }
```

Set up the SVG and run the raincloud plot function several times.

```javascript
  const svg = d3.select('main')
    .call(dom)
    .select('svg')
```

### Display to Table

Get values by math.js and display them to table.

```javascript
plotA = document.getElementById('plotA');
plotA.addEventListener('click', () => {
    const arrdata = dataA;
    const min = math.min(arrdata);
    const median = math.median(arrdata);
    const mean = math.mean(arrdata);
    const max =math.max(arrdata);
    const fquartile = math.quantileSeq(arrdata, 0.25);
    const tquartile = math.quantileSeq(arrdata, 0.75);
    const tabletext = 
        `<td>${min}</td>
        <td>${fquartile}</td>
        <td>${median}</td>
        <td>${mean}</td>
        <td>${tquartile}</td>
        <td>${max}</td>`;
    document.getElementById('valueTable').innerHTML = tabletext;
})
```

Close the anonymous function opened at the very beginning.

```javascript
})()
```