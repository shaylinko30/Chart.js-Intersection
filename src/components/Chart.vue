<template>
  <div class="chart">
    <div class="intersection">
      <p class="showData"></p>
    </div>
    <canvas id="canvas"></canvas>
  </div>
</template>

<script>
import Chart from 'chart.js'

export default {
  name: 'Chart',
  mounted () {
    this.chartData()
  },
  data () {
    return {
      // Comment and uncomment  firstLine and secondLine to see the 
      // intersection point graphed in different parts of the chart
      firstLine: [1000, 4000, 7000, 10000],
      secondLine: [0, 5000, 10000, 15000],
      // firstLine: [30000, 45000, 60000, 75000],
      // secondLine: [0, 40000, 80000, 120000],
      // firstLine: [3250, 3500, 3750, 4000],
      // secondLine: [1100, 2100, 3100, 4100],
      chartWidth: 0,
      chartHeight: 0,
      bufferX: 0,
      bufferY: 0,
      // Don't divide by zero
      xTickMax: 1.0,
      yTickMax: 1.0,
      dotDiameter: 10
    }
  },
  methods: {
    getIntersection () {
      // slope = (y2 - y1) / (x2 - x1)
      // If x does not start at zero, minus x1 from x2
      // Issue: The denominator is hard-coded so that the Subtitle displays correctly
      const firstSlope = (this.firstLine[this.firstLine.length - 1] - this.firstLine[0]) / 3 // this.xTickMax
      const secondSlope = (this.secondLine[this.secondLine.length - 1] - this.secondLine[0]) / 3 // this.xTickMax
      // set y = y ( remember that y = mx + b => mx + b = nx + a => (b - a) / (n - m))
      const xIntersect = (this.firstLine[0] - this.secondLine[0]) / (secondSlope - firstSlope)
      // y = mx + b substitute in x
      const yIntersect = secondSlope * xIntersect + this.secondLine[0]
      // x-axis is in years convert to days (365 days/ 1 year)
      const days = xIntersect * 365.25
      return [days, yIntersect]
    },
    setIntersection () {
      const [intersection, ...restOfIntersection] = this.$el.getElementsByClassName('intersection')
      const [para, ...restOfPara] = this.$el.getElementsByClassName('showData')
      const [ x, y ] = this.getIntersection()
      para.innerText = ` ≈ ${Math.round(x)} days \n $${y.toLocaleString(undefined, { maximumFractionDigits: 2 })}`
      // times point by y and x scales
      const xScale = (x / 365.25) / this.xTickMax
      // y scale is backwards (ex: pixels 369 -> 0 and chart 0 -> 16,000)
      const yScale = (this.yTickMax - y) / this.yTickMax
      let newX = this.chartWidth * xScale + this.bufferX
      let newY = this.chartHeight * yScale + this.bufferY
      // Center dot
      newX -= this.dotDiameter / 2
      newY -= this.dotDiameter / 2
      intersection.style.left = `${newX}px`
      intersection.style.top = `${newY}px`
    },
    chartData () {
      const datasets = [{
        label: 'Intersection',
        backgroundColor: 'rgba(100, 65, 100, 1)'  
      },
      {
        label: 'First Line',
        data: this.firstLine,
        backgroundColor: 'rgba(100, 100, 100, 0.75)'
      },
      {
        label: 'SecondLine',
        data: this.secondLine,
        backgroundColor: 'rgba(65, 100, 100, 0.75)' 
      }]
      const canvas = this.$el.querySelector('#canvas')
      const chart = new Chart(canvas, {
        type: 'line',
        data: {
          labels: ['0yr', '1yr', '2yr', '3yr']
        },
        options: {
          title: {
            display: true,
            fontSize: 20,
            // Title and subtitle
            text:
              [
                'Two Lines',
                // Issue: This is run before xTickMax is set, so the x intersect will be inaccurate without hard-coding x2 - x1 in getIntersection
                `Intersection: $${this.getIntersection()[1].toLocaleString(undefined, { maximumFractionDigits: 2 })} at ${Math.round(this.getIntersection()[0])} days`
              ]
          },
          maintainAspectRatio: false,
          scales: {
            yAxes: [{
              ticks: {
                callback: (value, index, values) => {
                  return '$' + value
                },
                // The math will need to be modified if the y-axis does not start at 0
                beginAtZero: true,
                responsive: true,
                maintainAspectRatio: true
              }
            }]
          }
        }
      })
      chart.data.datasets = datasets
      chart.update()
      // Set the (0,0) coordinate
      this.bufferX = chart.chartArea.left
      this.bufferY = chart.chartArea.top
      this.chartWidth = chart.chartArea.right - chart.chartArea.left
      this.chartHeight = chart.chartArea.bottom - chart.chartArea.top
      // This will need to be modified if x-axis does not increment by 1
      // Follow the format of the yTickMax to extract the x-axis max
      this.xTickMax = chart.scales['x-axis-0'].ticks.length - 1
      this.yTickMax = Number(chart.scales['y-axis-0'].ticks[0].replace(/[^0-9.]/g, ''))
      this.setIntersection()
    }
  }
}
</script>

<style>
body {
  margin: auto;
}
.intersection {
  height: 10px;
  width: 10px;
  background: rgba(100, 65, 100, 1);
  position: absolute;
  border-radius: 100%;
  border: rgba(100, 65, 100, 1);
  opacity: 0.95;
}
.showData {
  position: relative;
  display: none;
  padding: 0em 0em 0em 1em;
  font-size: 1em;
  background: #101010;
  color: #ffffff;
  padding: 0.5em;
  margin-left: 0.75em;
  border-radius: 10%;
}
.intersection:hover .showData {
  display: inline-block;
}
.chart {
  position: relative;
  margin: auto;
  /* The height and width can be changed (refresh the page if changed when app is running) */
  height: 400px;
  width: 900px;
}
</style>