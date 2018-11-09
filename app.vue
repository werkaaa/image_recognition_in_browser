<template>
    <div id="app">

        <div class="note">
            {{ status }}
        </div>

        <div class="canvas-container">
            <canvas id="input-canvas"
                    width="140"
                    height="140"
                    @mousemove="draw"
            />
        </div>

        <button id="clear_value" @click="clear">Clear</button>
        <button id="inference" @click="inference">
            Inference
        </button>

        <table>
            <tr>
                <th>Digits</th>
                <th>Probability</th>
            </tr>
            <tr v-for="(pred, index) in preds">
                <td>{{ index }}</td>
                <td>{{ readable(pred) }}</td>
            </tr>
        </table>

    </div>
</template>

<script>
    import Predictor from './predictor';
    const MODEL_URL = 'model/model.json';
    //const WEIGHTS_URL = 'weights_manifest.json';
    const HEIGHT = 28;
    const WIDTH = 28;
    const pixels = [];
    for (let i = 0; i < HEIGHT * WIDTH; i++) {
        pixels[i] = 0;
    }
    const preds = [];
    for (let i = 0; i < 10; i++) {
        preds[i] = 0;
    }
    export default {
        name: 'app',
        components: {},
        data: function () {
            return {
                preds: preds,
                predictor: new Predictor(),
                pixels: pixels,
                status: "Loading...",
            }
        },
        created: function() {
            this.predictor.load(MODEL_URL).then(() => {
                this.status = "Model loaded!";
            });
        },
        methods: {
            readable(x) {
                return x.toFixed(3);
            },
            draw(e) {
                if (e.buttons == 1) {
                    const ctx = document.getElementById('input-canvas').getContext('2d');
                    ctx.fillStyle = "rgb(0,0,0)";
                    ctx.fillRect(e.offsetX, e.offsetY, 8, 8);
                    const x = Math.floor(e.offsetY * 0.2);
                    const y = Math.floor(e.offsetX * 0.2) + 1;
                    for (let dy = 0; dy < 2; dy++){
                        for (let dx = 0; dx < 2; dx++){
                            if ((x + dx < WIDTH) && (y + dy < HEIGHT)) {
                                this.pixels[(y+dy) + (x+dx) * HEIGHT] = 1
                            }
                        }
                    }
                }
            },
            clear() {
                const ctx = document.getElementById('input-canvas').getContext('2d');
                ctx.fillStyle = "rgb(255,255,255)";
                ctx.fillRect(0, 0, 140, 140);
                for (var i = 0; i < 28*28; i++) {
                    this.pixels[i] = 0;
                }
            },
            inference() {
                this.predictor.predict(this.pixels).then((value) => {
                    this.preds = Array.prototype.slice.call(value);
                });
            }
        }
    }
</script>




<style >
    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: left;
        color: #2c3e50;
        margin-top: 60px;
    }
    .canvas-container {
        border: solid 2px #666;
        width: 144px;
        height: 144px;
    }
    table {
    }
</style>