<template>
  <div class="home">
    <GameCanvas :basis="basisFormat" :map-size="mapSize" :arr="lifeMap" />
    <div class="home-ui">
      X <input
        type="range"
        id="x"
        :min="-wMax"
        :max="wMax"
        step="1"
        v-model="basis.x"
      >
      <br>
      Y <input
        type="range"
        id="y"
        :min="-hMax"
        :max="hMax"
        step="1"
        v-model="basis.y"
      >
      <br>
      Zoom <input
        type="range"
        id="zoom"
        min="4"
        max="20"
        step="1"
        v-model="basis.zoom"
      >
      <br>
      <button @click="next">
        Next
      </button>
      <button @click="play">
        Play
      </button>
    </div>
  </div>
</template>

<script lang="ts">
/* eslint-disable radix */
/* eslint-disable no-plusplus */
/* eslint-disable no-param-reassign */
/* eslint-disable prefer-destructuring */
/* eslint-disable no-shadow */
/* eslint-disable no-continue */

import { defineComponent } from 'vue'
import GameCanvas from '@/components/GameCanvas.vue'

interface Basis {
  x: number,
  y: number,
  zoom: number,
}

export default defineComponent({
  name: 'Home',
  components: {
    GameCanvas
  },
  data () {
    return {
      basis: {
        x: '0',
        y: '0',
        zoom: '4'
      },
      mapSize: 80,
      lifeMap: [] as boolean[][],
      nextFrameMap: [] as boolean[][],
      interval: null as any
    }
  },
  computed: {
    basisFormat (): Basis {
      return {
        x: -parseInt(this.basis.x),
        y: -parseInt(this.basis.y),
        zoom: Number(this.basis.zoom)
      }
    },
    wMax (): number {
      return this.mapSize * this.basisFormat.zoom - window.innerWidth + 10
    },
    hMax (): number {
      return this.mapSize * this.basisFormat.zoom - window.innerHeight + 10
    }
  },
  created () {
    this.lifeMap = [...Array(this.mapSize)]
      .map(() => [...Array(this.mapSize)]
        .map(() => !!Math.round(Math.random())))
    this.nextFrameMap = [...Array(this.mapSize)]
      .map(() => [...Array(this.mapSize)]
        .map(() => false))
  },
  beforeUnmount () {
    if (this.interval) clearInterval(this.interval)
  },
  methods: {
    play () {
      this.interval = setInterval(this.next, 1)
    },
    next () {
      const { lifeMap: lMap, nextFrameMap: nfMap } = this
      for (let y = 0, yLen = lMap.length; y < yLen; y++) {
        for (let x = 0, xLen = lMap[y].length; x < xLen; x++) {
          nfMap[y][x] = this.calculateItem(lMap, x, y)
        }
      }
      this.lifeMap = nfMap
      this.nextFrameMap = lMap
    },
    calculateItem (lMap: boolean[][], x: number, y: number): boolean {
      const num = this.getNeighbors(lMap, x, y)
      /**
       * 邻居数量等于 3 时生命诞生
       * 邻居数量等于 2 时生命保持现状
       * 邻居数量小于 2 或者大于 3 时生命死亡
       */
      if (num === 3) return true
      if (num === 2) return !!lMap[y][x]
      return false
    },
    getNeighbors (lMap: boolean[][], x: number, y: number): number {
      let num = 0
      for (let nY = -1; nY < 2; nY++) {
        if (!lMap[nY + y]) continue
        for (let nX = -1; nX < 2; nX++) {
          // 不是自己，并且结果为生命（true）
          if ((nY || nX) && lMap[nY + y][nX + x]) num++
        }
      }
      return num
    }
  }
})
</script>

<style scoped lang="less">
.home {
  display: block;
  width: 100%;
  height: 100%;
  background: black;

  &-ui {
    position: fixed;
    z-index: 9999;
    bottom:5px;
    left: 5px;
    background-color: #ffffffad;
    padding: 10px;
    border-radius: 5px;
  }
}
</style>
