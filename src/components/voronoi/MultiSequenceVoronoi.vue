<template>
  <rect
    fill-opacity="0"
    :width="width"
    :height="height"
    @mousemove="voronoiMoveHandler"
    @click="voronoiClickHandler"
    @mouseleave="outHandler" />
</template>
<script>
import { Delaunay } from 'd3-delaunay'

export default {
  props: {
    // data
    series: { type: Array, required: true },

    // data accessors
    x: { type: Function, required: true },
    y: { type: Function, required: true },

    // data scaling
    xScale: { type: Function, required: true },
    yScale: { type: Function, required: true },

    // dimensions
    width: { type: Number, required: true },
    height: { type: Number, required: true },

    // handlers
    moveHandler: { type: Function, default: () => {} },
    outHandler: { type: Function, default: () => {} },
    clickHandler: { type: Function, default: () => {} },

    // re-computation handling
    recompute: { type: Number, default: 0 },
    recomputeVoronoi: { type: Boolean, default: true }
  },
  watch: {
    recompute () {
      if (this.recomputeVoronoi) return
      this.computedDelaunay = this.delaunay
    }
  },
  data () {
    return {
      computedDelaunay: {}
    }
  },
  beforeMount () {
    if (this.recomputeVoronoi) return
    this.computedDelaunay = this.delaunay
  },
  computed: {
    proxyDelaunay () {
      return this.recomputeVoronoi ? this.delaunay : this.computedDelaunay
    },
    data () {
      return this.series.reduce((rec, d, i) => {
        return rec.concat(d.map((point, j) => {
          return {
            ...point,
            indexSeries: i,
            indexPoints: j
          }
        }))
      }, [])
    },
    delaunay () {
      return Delaunay.from(this.data, d => this.xScale(this.x(d)), d => this.yScale(this.y(d)))
    }
  },
  methods: {
    getOffset (e) {
      const { x, y } = e.target.getBoundingClientRect()
      return this.proxyDelaunay.find(e.x - x, e.y - y)
    },
    voronoiClickHandler (e) {
      this.clickHandler(this.data[this.getOffset(e)])
    },
    voronoiMoveHandler (e) {
      this.moveHandler(this.data[this.getOffset(e)])
    }
  }
}
</script>
