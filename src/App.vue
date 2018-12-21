<template>
    <div id="q-app">
        <div class="container">
            <Canvsius></Canvsius>
        </div>
    </div>
</template>

<script>
    import Canvsius from './layouts/CanvasComponent.vue'
    import trainData from './assets/mnistTrain.json'
    // quasar dev -m cordova -T android

    export default {
        name: 'App',
        components: {
            Canvsius
        },
        data() {
            return {
                msg: 'qqq',
            }
        }
    }
</script>

<style>
    @import "~bootstrap/dist/css/bootstrap.min.css";
</style>
