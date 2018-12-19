<template>
    <div id="q-app">
        <div class="container-fluid">
            <div class="d-flex justify-content-start flex-column">
                <Canvsius></Canvsius>
                <p>message: {{msg}}</p>
                <button type="button" class="btn-info" style="width: 100px" @click="msg='dima is a black dude'">кнопка
                </button>
            </div>
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
