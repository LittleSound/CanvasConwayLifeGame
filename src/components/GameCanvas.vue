<template>
  <canvas ref="gameCanvasDom" class="gamecanvas" :width="canvasWidth" :height="canvasHeight">
    Your browser does not support "canvas", please upgrade your browser.
  </canvas>
</template>

<script lang="ts">
/* eslint-disable no-plusplus */
/* eslint-disable no-param-reassign */
/* eslint-disable prefer-destructuring */
/* eslint-disable no-shadow */

import {
  defineComponent, onMounted, toRefs, reactive, ref, watch, onBeforeUnmount, nextTick,
} from 'vue';

export default defineComponent({
  name: 'GameCanvas',
  props: {
    basis: {
      type: Object,
      default: () => ({
        x: 0,
        y: 0,
        zoom: 10,
      }),
    },

    /** 需要渲染的数组 */
    arr: {
      type: Array,
      required: true,
    },
    /** 边距 */
    margin: {
      type: Number,
      default: 1,
    },
    /** 生命颜色 */
    lifeColor: {
      type: String,
      default: '#42b983',
    },
    /** 死亡颜色 */
    deathColor: {
      type: String,
      default: '#0f2b1f',
    },
    /** 边框颜色 */
    borderColor: {
      type: String,
      default: '#000',
    },
    /** 地图尺寸 */
    mapSize: {
      type: Number,
      default: 50,
    },
    /** 绘制时间间隔 */
    drawInterval: {
      type: Number,
      default: 17,
    },
  },
  setup(props) {
    const {
      basis, margin, lifeColor, deathColor, borderColor, drawInterval, arr,
    } = toRefs(props);
    /** 绑定到 canvas 的 ref 上面获取 DOM */
    const gameCanvasDom: any = ref(null);
    const canvasWidth = ref(300);
    const canvasHeight = ref(300);
    const obj = reactive({
      ctx: null as any,
    });
    /** 节流，传入需要节流的方法和毫秒时间 */
    function throttling(func: (...p: any[]) => any, time: number) {
      let throttlingSwitch = true;
      let args: any[] = [];
      return function (...newArgs: any[]) {
        // 执行时采用最新的参数
        args = newArgs;
        if (throttlingSwitch) {
          throttlingSwitch = false;
          setTimeout(() => {
            throttlingSwitch = true;
            func(...args);
          }, time);
        }
      };
    }

    /**
     * 坐标变换
     */
    function transform(x: number, y: number) {
      const centerW = Math.round(window.innerWidth / 2);
      const centerH = Math.round(window.innerHeight / 2);
      const sizeHalf = Math.round(arr.value.length * basis.value.zoom) / 2;
      // 位置 x, 位置 y, 宽度，高度
      return [
        Math.round(basis.value.zoom * x) + centerW - sizeHalf + basis.value.x,
        Math.round(basis.value.zoom * y) + centerH - sizeHalf + basis.value.y,
        basis.value.zoom,
        basis.value.zoom,
      ];
    }

    /**
     * 绘制矩阵
     */
    function drawMatrix(ctx: any, arr: Array<Array<any>>) {
      ctx.clearRect(0, 0, canvasWidth.value, canvasHeight.value);
      for (let y = 0, yLen = arr.length; y < yLen; y++) {
        for (let x = 0, xLen = arr[y].length; x < xLen; x++) {
          ctx.fillStyle = arr[y][x] ? lifeColor.value : deathColor.value;
          ctx.fillRect(...transform(x, y));
          if (basis.value.zoom > 5) ctx.strokeRect(...transform(x, y));
        }
      }
    }

    /** 带节流的绘制矩阵方法 */
    const drawMatrixFromThrottling = throttling(drawMatrix, drawInterval.value);

    /** 窗口缩放时调整画布尺寸 */
    const resizeCanvas = throttling(() => {
      canvasWidth.value = window.innerWidth;
      canvasHeight.value = window.innerHeight;
      nextTick(() => {
        drawMatrix(obj.ctx, arr.value as any);
      });
    }, 16);

    onMounted(() => {
      if (!gameCanvasDom.value.getContext) return;
      obj.ctx = gameCanvasDom.value.getContext('2d');
      // 边框颜色和宽度
      obj.ctx.lineWidth = margin.value;
      obj.ctx.strokeStyle = borderColor.value;

      window.addEventListener('resize', resizeCanvas, false);
      resizeCanvas();
    });

    onBeforeUnmount(() => {
      window.removeEventListener('resize', resizeCanvas, false);
    });

    watch([basis, arr], () => {
      console.log('arr:', arr.value);
      drawMatrixFromThrottling(obj.ctx, arr.value);
    }, { deep: true });

    return {
      gameCanvasDom,
      canvasWidth,
      canvasHeight,
    };
  },
});
</script>

<style scoped lang="less">
.gamecanvas {
  display: block;
}
</style>
