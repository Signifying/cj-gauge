# cj-gauge
```js
const chartData = {
  categories: [
    {
      value: 4,
      color: "#F0F8FF",
      color2: "#F0F8FF",
    },
    {
      value: 1,
      color: "#87CEFA",
      color2: "#1E90FF",
    },
  ],
  series: [
    {
      name: "分数",
      data: 0.72,
    },
  ],
};
const opts = {
  pix: 1,
  width: 375,
  height: 250,
  fontSize: 10,
  gauge: {
    width: 15,
    width2: 5,
    labelColor: "#1E90FF",
    labelOffset: 30,
    labelFontSize: 10,
    neme: "标准",
    nemeOffset: 0,
    nemeColor: "#4682B4",
    arrowsColor: "#4169E1",
    startAngle: 0.85,
    endAngle: 0.15,
    startNumber: 50,
    endNumber: 100,
    standard: 90,
  },
  splitLine: {
    fixRadius: 0,
    splitNumber: 10,
    width: 6,
    color: "#B0C4DE",
    lineWidth: 3,
    childLineWidth: 3,
    childNumber: 5,
    childWidth: 3,
  },
  pointer: {
    width: 45,
    color: "#B0C4DE",
    shadowColor: "#6495ED",
    shadowBlur: 2,
    line: 1,
    lineColor: "#EAFFFD",
  },
  tooltip: {
    fontSize: 14,
    fontOffset: 0,
    valueOffset: 0,
    fontColor: "#6495ED",
    gradientColor: ["#1E90FF", "#00008B"],
  },
};
```