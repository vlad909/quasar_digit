<template>
  <div id="q-app">
    <Canvsius></Canvsius>
    <p>message: {{msg}}</p>
    <button type="button" @click="msg='dima is a black dude'">кнопка</button>
  </div>
</template>

<script>
  import Canvsius from './layouts/CanvasComponent.vue'
  // quasar dev -m cordova -T android

  export default {
    name: 'App',
    components: {
      Canvsius
    },
    data() {
      return {
        msg: 'qqq'
      }
    }
  }
</script>

<style>
</style>
