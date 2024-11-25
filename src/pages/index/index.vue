<template>
  <view>
    <cj-gauge :chartData="chartData" :opts="opts"></cj-gauge>
  </view>
</template>

<script>
import cjGauge from "../../components/cj-gauge/cj-gauge.vue";

export default {
  components: { cjGauge },
  data() {
    return {
      chartData: {},
      opts: {
        pointer: {
          width: 50,
        },
      },
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      setTimeout(() => {
        this.chartData = {
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
      }, 1000);
    },
  },
};
</script>

<style>
</style>