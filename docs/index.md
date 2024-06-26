---
toc: false
---

# San Francisco wind

This example accesses near real-time wind data using R and Python data loaders, and uses GitHub actions to automatically rebuild and deploy to GitHub pages every 2 hours.

The R and Py data loaders are redundant; this is a pedagogical example to test GH actions for both.

**Data:** NOAA Tides and Currents, [CO-OPS API](https://api.tidesandcurrents.noaa.gov/api/prod/)

## Using data from R data loader

Data loader: `docs/data/windR.csv.R`

```js echo
const windR = await FileAttachment("./data/windR.csv").csv({ typed: true });
```

```js
// Direction abbreviations in meaninful order
const directions = d3
  .groupSort(
    windR,
    (g) => d3.median(g, (d) => d.windDirAngle),
    (d) => d.windDirAbb
  )
  .reverse();
```

```js
function windyRChart(width) {
  return Plot.plot({
    width: width,
    marginRight: 40,
    color: { range: ["skyblue", "#008BFF"], interpolate: "hcl" },
    x: { grid: true, label: "Date" },
    y: { grid: true, label: "Wind speed (knots)" },
    marks: [
      Plot.areaY(windR, {
        x: "Date Time",
        y: "Speed",
        fill: "skyblue",
        opacity: 0.3,
      }),
      Plot.line(windR, {
        x: "Date Time",
        y: "Speed",
        stroke: "skyblue",
        strokeWidth: 1,
      }),
      Plot.dot(windR, {
        x: "Date Time",
        y: "Speed",
        fill: "Speed",
        opacity: 0.8,
      }),
      Plot.vector(windR, {
        x: "Date Time",
        y: "Speed",
        length: "Speed",
        rotate: (d) => 180 + d.windDirAngle,
        stroke: "Speed",
        anchor: "start",
        tip: true,
        title: (d) =>
          `Wind speed: ${d.Speed} knots\nWind direction: ${d.windDirAbb}`,
      }),
    ],
  });
}
```

## Using data from Python data loader

Data loader: `docs/data/windPy.csv.py`

```js echo
const windPy = await FileAttachment("./data/windPy.csv").csv({ typed: true });
```

```js
function windyPyChart(width) {
  return Plot.plot({
    width: width,
    x: { domain: directions, label: "Wind direction" },
    marks: [
      Plot.barY(
        windPy,
        Plot.groupX({ y: "count" }, { x: "windDirAbb", fill: "#008BFF" })
      ),
      Plot.ruleY([0]),
    ],
  });
}
```

<div class="grid grid-cols-4">
  <div class="card grid-colspan-4">${resize(width => windyRChart(width))}</div>
</div>

<div class="grid grid-cols-4">
  <div class="card grid-colspan-2">${Inputs.table(windPy, {rows: 17.5})}</div>
  <div class="card grid-colspan-2">${resize(width => windyPyChart(width))}</div>
</div>
