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
      const start = this.hexToRGB(this.from);
      const end = this.hexToRGB(this.to);

      const average = end.map((val, i) => ((val - start[i]) / count));
      
      for (let i = 0; i <= count; i++) {
        let temp = start.map((val, k) => Math.round(val + (average[k] * i)));
        this.colors.push(this.generateOutput(temp));
      }
    },

    hexToRGB(hex) {
      return (hex = hex.replace('#', '')).match(new RegExp('(.{'+hex.length/3+'})', 'g')).map(function(l) { return parseInt(hex.length%2 ? l+l : l, 16) })
    },

    RGBToHex(rgb) {
      return '#' + rgb.map(x => {
        const hex = x.toString(16)
        return hex.length === 1 ? '0' + hex : hex
      }).join('')
    },

    generateOutput(RGBArray) {
      return this.RGBToHex(RGBArray);
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
			if(color.rgba.a == 1) {
				this.from = color.hex;
			}
			else {
				this.from = 'rgba(' + color.rgba.r + ', ' + color.rgba.g + ', ' + color.rgba.b + ', ' + color.rgba.a + ')';
      }
      
      this.generateColor()
    },
    
    updateToPicker(color) {
			if(color.rgba.a == 1) {
				this.to = color.hex;
			}
			else {
				this.to = 'rgba(' + color.rgba.r + ', ' + color.rgba.g + ', ' + color.rgba.b + ', ' + color.rgba.a + ')';
      }
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
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
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
