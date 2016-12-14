<template>
  <g :transform="transform">
    <rect :width="w" :height="w"
          :x="0" :y="0"
          :style="{ fill: fill }" />

    <Pythagoras v-if="!stackExceeded" :w="nextLeft"
                x=0 :y="-nextLeft"
                :lvl="lvl + 1" :maxlvl="maxlvl"
                :height-factor="heightFactor"
                :lean="lean"
                :left="true"
                :right="false" />

    <Pythagoras v-if="!stackExceeded" :w="nextRight"
                :x="w - nextRight" :y="-nextRight"
                :lvl="lvl + 1" :maxlvl="maxlvl"
                :height-factor="heightFactor"
                :lean="lean"
                :left="false"
                :right="true" />

  </g>
</template>

<script>
import { interpolateViridis } from 'd3-scale'

Math.deg = radians => {
  return radians * (180 / Math.PI)
}

const memo = {}

const key = ({ w, heightFactor, lean }) => {
  return [w, heightFactor, lean].join('-')
}

const memoizedCalc = args => {
  const memoKey = key(args)

  if (memo[memoKey]) {
    return memo[memoKey]
  } else {
    const { w, heightFactor, lean } = args
    const trigH = heightFactor * w

    const result = {
      nextRight: Math.sqrt(trigH ** 2 + (w * (0.5 + lean)) ** 2),
      nextLeft: Math.sqrt(trigH ** 2 + (w * (0.5 - lean)) ** 2),
      A: Math.deg(Math.atan(trigH / ((0.5 - lean) * w))),
      B: Math.deg(Math.atan(trigH / ((0.5 + lean) * w)))
    }

    memo[memoKey] = result
    return result
  }
}

export default {
  name: 'Pythagoras-Component',
  props: ['w', 'x', 'y', 'heightFactor', 'lean', 'left', 'right', 'lvl', 'maxlvl'],

  data () {
    return {
      nextRight: 0,
      nextLeft: 0,
      rotate: ''
    }
  },

  watch: {
    heightFactor () {
      this.calculateRotate()
    }
  },

  methods: {
    calculateRotate () {
      const { nextRight, nextLeft, A, B } = memoizedCalc({
        w: this.w,
        heightFactor: this.heightFactor,
        lean: this.lean
      })

      this.nextRight = nextRight
      this.nextLeft = nextLeft

      if (this.left) {
        this.rotate = `rotate(${-A} 0 ${this.w})`
      } else if (this.right) {
        this.rotate = `rotate(${B} ${this.w} ${this.w})`
      }
    }
  },

  computed: {
    transform () {
      return `translate(${this.x} ${this.y}) ${this.rotate}`
    },

    fill () {
      return interpolateViridis(this.lvl / this.maxlvl)
    },

    stackExceeded () {
      return this.lvl >= this.maxlvl || this.w < 1
    }
  }
}
</script>
