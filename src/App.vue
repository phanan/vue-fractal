<template>
  <div id="app">
    <div class="App">
      <p class="App-intro">
        <svg :width="svg.width" :height="svg.height" ref="svg">
            <Pythagoras :w="baseW"
                        :h="baseW"
                        :heightFactor="heightFactor"
                        :lean="lean"
                        :x="svg.width / 2 - 40"
                        :y="svg.height - baseW"
                        :lvl="0"
                        :maxlvl="currentMax"/>
        </svg>
      </p>
  </div>
  </div>
</template>

<script>
import { select as d3select, mouse as d3mouse } from 'd3-selection'
import { scaleLinear } from 'd3-scale'
import Pythagoras from './components/Pythagoras.js'

const throttleWithRAF = fn => {
  let running = false
  return function () {
    if (running) return
    running = true
    const throttlerFn = 'requestIdleCallback' in window ? window.requestIdleCallback : window.requestAnimationFrame
    throttlerFn(() => {
      fn.apply(this, arguments)
      running = false
    })
  }
}

const realMax = 11

export default {
  name: 'app',

  components: { Pythagoras },

  data () {
    return {
      svg: {
        width: 1280,
        height: 600
      },
      currentMax: 0,
      baseW: 80,
      heightFactor: 0,
      lean: 0
    }
  },

  mounted () {
    d3select(this.$refs.svg).on('mousemove', this.onMouseMove)
    this.next()
  },

  methods: {
    next () {
      if (this.currentMax < realMax) {
        this.currentMax += 1
        setTimeout(this.next, 500)
      }
    },

    onMouseMove () {
      const [x, y] = d3mouse(this.$refs.svg)
      this.update(x, y)
    },

    update: throttleWithRAF(function (x, y) {
      const scaleFactor = scaleLinear()
        .domain([this.svg.height, 0])
        .range([0, 0.8])
      const scaleLean = scaleLinear()
        .domain([0, this.svg.width / 2, this.svg.width])
        .range([0.5, 0, -0.5])
      this.heightFactor = scaleFactor(y)
      this.lean = scaleLean(x)
    })
  }
}
</script>

<style>
.App {
  text-align: center;
}

.App-logo {
  animation: App-logo-spin infinite 20s linear;
  height: 80px;
}

.App-header {
  background-color: #222;
  height: 150px;
  padding: 20px;
  color: white;
}

.App-intro {
  font-size: large;
}

@keyframes App-logo-spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
</style>
