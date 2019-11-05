<template>
    <div class="color-wrapper">
        <div class="color-header">
            <div class="input-group-append">
                <div class="color-picker" ref="colorpicker">
                    <span class="current-color" :style="'background-color: ' + from" @click="toogleFromPicker()" />
                    <span class="current-color" :style="'background-color: ' + to" @click="toogleToPicker()" />
                    <chrome-picker v-model="from" v-if="displayFromPicker" @input="updateFromPicker" />
                    <chrome-picker v-model="to" v-if="displayToPicker" @input="updateToPicker" />
                </div>
                <div class="count"><input type="number" v-model="count" min=1 @change="generateColor()" /></div>
            </div>
        </div>
        <div class="colors" >
            <div v-for="color in colors" :key="color" class="color" :style="{ backgroundColor: color }">
                <span class="color-name">{{color}}</span>
                <span class="copy" v-clipboard:copy="color"
                      v-clipboard:success="done">{{copyText}}</span>
            </div>
        </div>
    </div>

</template>

<script>
    import Vue from 'vue'
    import { Chrome } from 'vue-color';
    import VueClipboard from 'vue-clipboard2';

    Vue.use(VueClipboard)

    Vue.config.productionTip = false
    export default {
        name: 'app',
        components: {
            'chrome-picker': Chrome,
        },
        data() {
            return {
                from: '#2b2928',
                to: '#ab8077',
                count: 4,
                colors: ["#2b2928", "#453a38", "#5e4c48", "#785d57", "#916f67", "#ab8077"],
                displayFromPicker: false,
                displayToPicker: false,
                copy: false,
                copyText: "COPY"
            }
        },

        methods: {
            done: function () {
                this.copyText = "COPIED!"
                setTimeout(() => {
                    this.copyText = "COPY"
                }, 700)
            },

            generateColor() {
                this.colors = []
                let count = parseInt(this.count)+1;
                const start = this.hexToRGBA(this.from);
                const end = this.hexToRGBA(this.to);

                const average = end.map((val, i) => ((val - start[i]) / count));

                for (let i = 0; i <= count; i++) {
                    let temp = start.map((val, k) => k < 3 ? Math.round(val + (average[k] * i)) : (val + (average[k] * i)).toFixed(3));
                    this.colors.push(this.RGBAToHex(temp));
                }
            },

            hexToRGBA(hex) {
              hex = hex.charAt(0) === '#' ? hex.slice(1) : hex;
              return [
                parseInt(hex.slice(0, 2), 16),
                parseInt(hex.slice(2, 4), 16),
                parseInt(hex.slice(4, 6), 16),
                +((parseInt(hex.slice(6,8) || 'ff', 16) / 255).toFixed(2)),
              ]
            },

            RGBAToHex(rgba) {
                return '#' + rgba.map((x,i) => {
                    const hex = (i < 3 ? x : Math.floor(x * 255)).toString(16)
                    return hex.length === 1 ? '0' + hex : hex
                }).join('')
            },

            toogleFromPicker() {
                this.displayFromPicker ? this.hideFromPicker() : this.showFromPicker();
            },

            toogleToPicker() {
                this.displayToPicker ? this.hideToPicker() : this.showToPicker();
            },

            showFromPicker() {
                document.addEventListener('click', this.documentClick);
                this.displayFromPicker = true;
                this.displayToPicker = false;
            },
            hideFromPicker() {
                document.removeEventListener('click', this.documentClick);
                this.displayFromPicker = false;
            },

            hideToPicker() {
                document.removeEventListener('click', this.documentClick);
                this.displayToPicker = false;
            },

            showToPicker() {
                document.addEventListener('click', this.documentClick);
                this.displayToPicker = true;
                this.displayFromPicker = false;
            },

            updateFromPicker(color) {
                this.from = color.hex8;
                this.generateColor()
            },

            updateToPicker(color) {
                this.to = color.hex8;
                this.generateColor()
            },

            documentClick(e) {
                var el = this.$refs.colorpicker,
                    target = e.target;
                if(el !== target && !el.contains(target)) {
                    this.hideFromPicker()
                    this.hideToPicker()
                }
            }
        },
    }
</script>

<style>

    body {
        margin: 0;
    }

    .colors {
        display: flex;
        flex-direction: row;
        overflow: scroll;
        text-align: center;
        height: calc(100vh - 50px);
    }

    .color {
        width: 1000px;
        min-width: 200px;
        text-align: center;
        padding-top: 40px;
        cursor:pointer;
        justify-content: center;
    }

    .color .copy {
        display: none;
        margin-top: 30px;
        width: 80px;
        height: 20px;
        border: 1px solid gray;
        border-radius: 2px;
        color: white;
        padding: 3px;
        margin: auto;
        color: rgba(255,255,255,0.7);
        background-color: rgba(255,255,255,0.08);
        border-color: rgba(255,255,255,0.2);
        border-style: solid;
        border-width: 1px;
        border-radius: 0.3rem;
        transition: color 0.2s, background-color 0.2s, border-color 0.2s;
    }

    .color .copy:hover {
        color: rgba(255,255,255,0.8);
        text-decoration: none;
        background-color: rgba(255,255,255,0.2);
        border-color: rgba(255,255,255,0.3);
    }

    .color .color-name {
        padding: 6px;
        margin: auto;
        color: white;
        background-color: black;
        border-color: rgba(255,255,255,0.2);
        border-style: solid;
        border-width: 1px;
        border-radius: 0.3rem;
    }
    .color:hover .copy{
        display: block;
        margin-top: 30px;
    }

    .current-color {
        width: 30px;
        height: 30px;
        display: inline-block;
        margin-right: 5px;
        border-radius: 5px;
    }

    .color-header {
        width: 100%;
        height: 50px;
        position: relative;
        z-index: 2;
        justify-content: center;
        align-items: center;
    }

    .input-group-append {
        display: flex;
        z-index: 2;
        position: relative;
    }

    .count {
        height: 20px;
        margin-top: 10px;
        margin-left: 10px;
    }

    .count input {
        width: 35px;
        height: 25px;
        margin-right: 5px;
        border-radius: 5px;
    }

    .color-picker {
        position: relative;
        top: 10px;
        display: inline-block;
        width: 70px;
        margin-left: 10px;
    }
</style>
