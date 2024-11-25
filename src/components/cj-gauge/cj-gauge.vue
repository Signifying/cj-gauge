<template>
  <canvas
    :id="canvasId"
    :canvasId="canvasId"
    :chartData="chartData"
    :style="{
      width: cWidth + 'px',
      height: cHeight + 'px',
    }"
    :opts="opts"
  />
</template>


<script>
const deepMerge = (obj1, obj2) => {
  let result = {};
  // 遍历obj1的属性
  for (let key in obj1) {
    if (obj1.hasOwnProperty(key)) {
      result[key] =
        typeof obj1[key] === "object" && obj1[key] !== null
          ? deepMerge(obj1[key], {})
          : obj1[key];
    }
  }
  // 遍历obj2的属性
  for (let key in obj2) {
    if (obj2.hasOwnProperty(key)) {
      if (typeof obj2[key] === "object" && obj2[key] !== null) {
        result[key] = deepMerge(result[key] || {}, obj2[key]);
      } else {
        result[key] = obj2[key];
      }
    }
  }
  return result;
};

export default {
  name: "CJGauge",
  props: {
    canvasId: {
      type: String,
      default: "cj_canvas",
    },
    cWidth: {
      type: Number,
      default: 375,
    },
    cHeight: {
      type: Number,
      default: 250,
    },
    background: {
      type: String,
      default: "none",
    },
    chartData: {
      type: Object,
      default: () => {},
    },
    opts: {
      type: Object,
      default: () => {},
    },
  },
  watch: {
    chartData: {
      handler(val, oldval) {
        this.darw();
      },
      immediate: false,
      deep: true,
    },
  },
  methods: {
    darw() {
      const ctx = uni.createCanvasContext(this.canvasId, this);
      this.drawGauge(
        this.chartData.categories,
        this.chartData.series,
        this.opts,
        ctx
      );
    },
    drawGauge(categories, series, opts, context) {
      opts = deepMerge(
        {
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
        },
        opts
      );
      const gaugeOption = opts.gauge;
      const totalAngle = 2 - gaugeOption.startAngle + gaugeOption.endAngle;
      categories = this.getCategoriesAngle(
        categories,
        gaugeOption.startAngle,
        totalAngle
      );
      var centerPosition = {
        x: opts.width / 2,
        y: opts.height / 2,
      };
      const innerRadius =
        Math.min(centerPosition.x, centerPosition.y) -
        10 -
        gaugeOption.width -
        gaugeOption.width2;
      // 背景
      for (let i = 0; i < categories.length; i++) {
        let eachCategories = categories[i];
        context.beginPath();
        context.setLineWidth(gaugeOption.width);
        context.setStrokeStyle(eachCategories.color);
        context.arc(
          centerPosition.x,
          centerPosition.y,
          innerRadius,
          eachCategories._startAngle_ * Math.PI,
          eachCategories._endAngle_ * Math.PI,
          false
        );
        context.stroke();
        context.beginPath();
        context.setLineWidth(gaugeOption.width2);
        context.setStrokeStyle(eachCategories.color2);
        context.arc(
          centerPosition.x,
          centerPosition.y,
          innerRadius + 5,
          eachCategories._startAngle_ * Math.PI,
          eachCategories._endAngle_ * Math.PI,
          false
        );
        context.stroke();
      }
      context.save();
      // 刻度线
      let splitAngle = totalAngle / opts.splitLine.splitNumber;
      let childAngle =
        totalAngle / opts.splitLine.splitNumber / opts.splitLine.childNumber;
      let radius =
        Math.min(centerPosition.x, centerPosition.y) -
        10 -
        opts.splitLine.lineWidth;
      let startX = -radius - opts.splitLine.fixRadius;
      let endX = -radius - opts.splitLine.fixRadius + opts.splitLine.width;
      let childendX =
        -radius - opts.splitLine.fixRadius + opts.splitLine.childWidth;
      context.translate(centerPosition.x, centerPosition.y);
      context.rotate((gaugeOption.startAngle - 1) * Math.PI);
      for (let i = 0; i < opts.splitLine.splitNumber + 1; i++) {
        context.beginPath();
        context.setStrokeStyle(opts.splitLine.color);
        context.setLineWidth(
          opts.splitLine.lineWidth * opts.pix || 2 * opts.pix
        );
        context.moveTo(startX, 0);
        context.lineTo(endX, 0);
        context.stroke();
        context.rotate(splitAngle * Math.PI);
      }
      context.restore();
      context.save();
      context.translate(centerPosition.x, centerPosition.y);
      context.rotate((gaugeOption.startAngle - 1) * Math.PI);
      for (
        let i = 0;
        i < opts.splitLine.splitNumber * opts.splitLine.childNumber + 1;
        i++
      ) {
        context.beginPath();
        context.setStrokeStyle(opts.splitLine.color);
        context.setLineWidth(
          opts.splitLine.childLineWidth * opts.pix || 1 * opts.pix
        );
        context.moveTo(startX, 0);
        context.lineTo(childendX, 0);
        context.stroke();
        context.rotate(childAngle * Math.PI);
      }
      context.restore();
      // 指针
      series = this.getSeriesAngle(series, gaugeOption, totalAngle);

      for (let i = 0; i < series.length; i++) {
        let eachSeries = series[i];
        context.save();
        context.translate(centerPosition.x, centerPosition.y);
        context.rotate(eachSeries._startAngle_ * Math.PI);
        context.beginPath();
        context.setLineWidth(opts.pointer.line);
        context.setStrokeStyle(opts.pointer.lineColor);
        context.arc(
          -innerRadius,
          0,
          (gaugeOption.width + gaugeOption.width2) / 2 -
            opts.pointer.line -
            opts.pointer.shadowBlur,
          0,
          2 * Math.PI,
          false
        );
        context.setFillStyle(gaugeOption.labelColor);
        context.shadowBlur = opts.pointer.shadowBlur;
        context.shadowColor = opts.pointer.shadowColor;
        context.fill();
        context.closePath();
        context.stroke();
        context.beginPath();
        context.arc(
          0,
          0,
          opts.pointer.width / 3,
          1.5 * Math.PI,
          0.5 * Math.PI,
          false
        );
        context.setFillStyle(gaugeOption.labelColor);
        context.moveTo(0, -opts.pointer.width / 3);
        context.quadraticCurveTo(
          -opts.pointer.width / 8,
          -(opts.pointer.width / 8) * 3,
          -(opts.pointer.width / 2 + opts.pointer.width / 8),
          0
        );
        context.moveTo(0, opts.pointer.width / 3);
        context.quadraticCurveTo(
          -opts.pointer.width / 8,
          (opts.pointer.width / 8) * 3,
          -(opts.pointer.width / 2 + opts.pointer.width / 8),
          0
        );
        context.lineTo(0, -opts.pointer.width / 3);
        context.closePath();
        context.fill();
        context.beginPath();
        context.setFillStyle("#FFFFFF");
        context.arc(0, 0, opts.pointer.width / 6, 0, 2 * Math.PI, false);
        context.fill();
        context.closePath();
        context.stroke();
        context.restore();
      }

      //标题
      let totalNumber = gaugeOption.endNumber - gaugeOption.startNumber;
      let splitNumber = totalNumber / opts.splitLine.splitNumber;
      let nowAngle = gaugeOption.startAngle;
      let nowNumber = gaugeOption.startNumber;
      radius -= gaugeOption.width / 2 + gaugeOption.labelOffset;
      radius = radius < 10 ? 10 : radius;
      context.translate(centerPosition.x, centerPosition.y);
      for (let i = 0; i < opts.splitLine.splitNumber + 1; i++) {
        var pos = {
          x: radius * Math.cos(nowAngle * Math.PI),
          y: radius * Math.sin(nowAngle * Math.PI),
        };
        var labelText = nowNumber;
        var startX1 = pos.x - 10;
        var startY2 = pos.y;

        if (
          (i === 0 || i === opts.splitLine.splitNumber) &&
          labelText !== gaugeOption.standard
        ) {
          context.beginPath();
          context.setFontSize(gaugeOption.labelFontSize);
          context.setFillStyle(gaugeOption.labelColor || opts.fontColor);
          context.fillText(
            (labelText / 100).toFixed(2),
            startX1,
            startY2 + gaugeOption.labelFontSize / 2
          );
          context.closePath();
          context.stroke();
        }

        if (labelText == gaugeOption.standard) {
          context.save();
          context.beginPath();
          context.setLineWidth(1);
          context.setStrokeStyle(gaugeOption.arrowsColor);
          context.rotate((-gaugeOption.endAngle + splitAngle * i) * Math.PI);
          context.moveTo(-Math.min(centerPosition.x, centerPosition.y) + 10, 0);
          context.lineTo(-Math.min(centerPosition.x, centerPosition.y), -2);
          context.lineTo(-Math.min(centerPosition.x, centerPosition.y), 2);
          context.lineTo(-Math.min(centerPosition.x, centerPosition.y) + 10, 0);
          context.setFillStyle(gaugeOption.arrowsColor);
          context.fill();
          context.closePath();
          context.stroke();
          context.restore();
          context.beginPath();
          context.setFontSize(gaugeOption.labelFontSize);
          context.setFillStyle(gaugeOption.labelColor || opts.fontColor);
          context.fillText(
            (labelText / 100).toFixed(2),
            startX1,
            startY2 + gaugeOption.labelFontSize / 2 - 5
          );
          context.setFillStyle(gaugeOption.nemeColor);
          context.fillText(
            gaugeOption.neme,
            startX1 - gaugeOption.nemeOffset,
            startY2 + gaugeOption.labelFontSize / 2 + 10
          );
          context.closePath();
          context.stroke();
        }

        nowAngle += splitAngle;
        if (nowAngle >= 2) {
          nowAngle = nowAngle % 2;
        }
        nowNumber += splitNumber;
      }

      // tooltip
      context.save();
      context.beginPath();
      context.setFontSize(opts.tooltip.fontSize);
      context.setFillStyle(opts.tooltip.fontColor);
      context.fillText(
        series[0].name,
        -context.measureText(series[0].name).width / 2 -
          opts.tooltip.fontOffset,
        opts.pointer.width - 10
      );
      const gradient = context.createLinearGradient(
        0,
        0,
        opts.pointer.width,
        0
      );
      gradient.addColorStop("0", opts.tooltip.gradientColor[0]);
      gradient.addColorStop("1", opts.tooltip.gradientColor[1]);

      context.fillStyle = gradient;
      // 绘制文本
      context.fillText(
        series[0].data.toFixed(2),
        -opts.tooltip.valueOffset -
          context.measureText(series[0].data.toFixed(2)).width / 2,
        opts.pointer.width + 10
      );
      context.closePath();
      context.stroke();
      context.restore();

      context.draw();
    },
    getCategoriesAngle(categories, startAngle, totalAngle) {
      let len = 0;
      let tempStartAngle = startAngle;
      categories.forEach((item) => {
        len += item.value;
      });
      for (let i = 0; i < categories.length; i++) {
        categories[i]._startAngle_ = tempStartAngle;
        if (totalAngle * (categories[i].value / len) + tempStartAngle > 2) {
          categories[i]._endAngle_ =
            totalAngle * (categories[i].value / len) + tempStartAngle - 2;
        } else {
          categories[i]._endAngle_ =
            totalAngle * (categories[i].value / len) + tempStartAngle;
        }
        tempStartAngle = categories[i]._endAngle_;
      }
      return categories;
    },
    getSeriesAngle(series, gaugeOption, totalAngle) {
      for (let i = 0; i < series.length; i++) {
        let temp =
          series[i].data <= gaugeOption.startNumber / gaugeOption.endNumber
            ? 0
            : (
                (series[i].data * gaugeOption.endNumber -
                  gaugeOption.startNumber) /
                (gaugeOption.endNumber - gaugeOption.startNumber)
              ).toFixed(2);
        series[i]._startAngle_ = -gaugeOption.endAngle + totalAngle * temp;
      }
      return series;
    },
  },
};
</script>

<style>
.content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.logo {
  height: 200rpx;
  width: 200rpx;
  margin: 200rpx auto 50rpx auto;
}

.text-area {
  display: flex;
  justify-content: center;
}

.title {
  font-size: 36rpx;
  color: #8f8f94;
}
</style>
