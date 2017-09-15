<template lang="pug">
div(class="chart")
  svg(width="800" height="400")
    g(id="container")
      rect(v-for="bar in bars" class="bar" v-bind:x="bar.x" v-bind:y="bar.y" v-bind:width="bar.width" v-bind:height="bar.height")
      g(class="axis axis--y")
      g(class="axis axis--x")

  pre {{ metrics }}
</template>

<script>
import * as d3 from 'd3';

export default {
  name: 'hello',
  created() {
    fetch('https://gist.githubusercontent.com/abarnhard/ceea34e440b9ec23a31019f600c9347b/raw/063afdb1f3e20fc3a5ff2999fe66d4a2dddcd1e8/sample-chart-data.json')
    .then(response => response.json())
    .then((data) => {
      this.metrics = data;
      this.drawGraph();
    });
  },
  mounted() {
    this.$svg = d3.select('svg');
    this.$chartContainer = this.$svg.select('g#container');

    const margin = { top: 20, right: 20, bottom: 20, left: 90 };
    this.width = +this.$svg.attr('width') - margin.left - margin.right;
    this.height = +this.$svg.attr('height') - margin.top - margin.bottom;

    this.xAxis = d3.scaleBand().rangeRound([0, this.width]).padding(0.1);
    this.yAxis = d3.scaleLinear().rangeRound([this.height, 0]);

    this.$chartContainer.attr('transform', `translate(${margin.left}, ${margin.top})`);
  },
  data() {
    return {
      metrics: [],
      $svg: null,
      $chartContainer: null,
      width: 0,
      height: 0,
      xAxis: null,
      yAxis: null,
    };
  },
  computed: {
    bars() {
      return this.metrics.map(metric => ({
        x: this.xAxis(metric.classCode),
        y: this.yAxis(metric.withinStandard + metric.outOfStandard),
        width: this.xAxis.bandwidth(),
        height: this.height - this.yAxis(metric.withinStandard + metric.outOfStandard),
      }));
    },
  },
  methods: {
    drawGraph() {
      this.xAxis.domain(this.metrics.map(metric => metric.classCode));
      this.yAxis.domain([
        0,
        d3.max(this.metrics, metric => metric.withinStandard + metric.outOfStandard),
      ]);

      const $xAxis = this.$chartContainer.select('g.axis.axis--x')
        .attr('transform', `translate(0, ${this.height})`);

      d3.axisBottom(this.xAxis)($xAxis);

      const $yAxis = this.$chartContainer.select('g.axis.axis--y');
      d3.axisLeft(this.yAxis).ticks(10)($yAxis);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}

.bar {
  fill: steelblue;
}
</style>
