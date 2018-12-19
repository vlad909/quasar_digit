<template>
  <canvas id="sheet"
          width="400px" height="400px"></canvas>
</template>
<script>
  import 'fabric'

  export default {
    data() {
      return {
        canvas: {}
      }
    },
    mounted() {
      this.canvas = new fabric.Canvas('sheet')
      this.canvas.isDrawingMode = true
      this.canvas.freeDrawingBrush.width = 10
      this.canvas.freeDrawingBrush.color = "#ff0000";
    }
  }
</script>
<style>
  #sheet {
    border: 1px black solid !important;
  }
</style>
