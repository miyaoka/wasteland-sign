<template>
  <div class="home" v-if="font">
    <input v-model="text" />
    <input type="range" min="0" max="1" step="any" v-model="rate" />
    <p>{{ `崩壊度： ${Math.floor(rate * 100)}%` }}</p>
    <svg width="500" height="500" xmlns="http://www.w3.org/2000/svg">
      <path
        fill="none"
        stroke="red"
        v-for="(stroke, i) in limitedStrokes"
        :key="i"
        :d="getD(stroke)"
      />
    </svg>
    <!-- <div v-for="(stroke, i) in limitedStrokes" :key="i">
      {{ stroke.map(command => command.type) }}
    </div> -->
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { Font, load } from 'opentype.js'

const fontURL =
  'https://fonts.gstatic.com/ea/notosansjapanese/v6/NotoSansJP-Medium.otf'

type MoveTo = {
  type: 'M'
  x: number
  y: number
}
type LineTo = {
  type: 'L'
  x: number
  y: number
}
type CurveTo = {
  type: 'C'
  x1: number
  x2: number
  y1: number
  y2: number
  x: number
  y: number
}
type ClosePath = {
  type: 'Z'
}
type Command = MoveTo | LineTo | CurveTo | ClosePath
type Path = {
  commands: Command[]
  fill: string
  stroke: null
  strokeWidth: number
}

const d = {
  M: ['x', 'y'],
  L: ['x', 'y'],
  C: ['x1', 'y1', 'x2', 'y2', 'x', 'y'],
  Z: [] as string[]
}
export default Vue.extend({
  name: 'home',
  data() {
    return {
      font: null as Font | null,
      text: 'ショッパーズ',
      rate: 0
    }
  },
  async mounted() {
    this.font = await this.loadFont()
  },
  computed: {
    path(): any | null {
      if (!this.font) return null
      return this.font.getPath(this.text, 0, 100, 72)
    },
    svg(): any {
      return this.path && this.path.toSVG()
    },
    commands(): Command[] {
      console.log(this.path)
      return this.path ? this.path.commands : []
    },
    strokes(): Command[][] {
      let index = 0
      return this.commands.reduce((acc: Command[][], command) => {
        const current = acc[index] || []
        current.push(command)
        acc[index] = current
        if (command.type === 'Z') index++
        return acc
      }, [])
    },
    limitedStrokes(): Command[][] {
      return this.strokes.filter(storke => Math.random() > this.rate)
    }
  },
  methods: {
    getD(cmds: Command[]): string {
      return cmds.reduce((acc, cmd) => {
        return (
          acc +
          ` ${cmd.type} ${d[cmd.type].map(p => Math.floor(cmd[p])).join(',')}`
        )
      }, '')
    },
    loadFont() {
      return new Promise<Font>((resolve, reject) => {
        load(fontURL, (err: string, font: Font) => {
          if (!font || err) {
            return reject(new Error(err))
          }
          return resolve(font)
        })
      })
    }
  }
})
</script>
<style lang="scss" scoped>
.home {
  padding: 30px;
}
input {
  font-size: 30px;
}
</style>
