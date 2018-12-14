<template>
  <div class="main animate" :style="{ 'background-color': colors.hex }" ref="target">
    <div class="color">
      <div class="title">
        <h1 class="animate" :style="{ 'color': inverse }">I Enjoy Lamp</h1>
        <p class="animate" :style="{ 'color': inverse }">
          Change the color of
          <a href="https://twitter.com/burkeholland">Burke's</a> LIFX connected lightbulb.
        </p>
        <p class="animate" :style="{ 'color': inverse }" id="livestreamLink">
          <a :href="livestream">Watch the Lamp Live Stream</a>
        </p>
      </div>
      <div class="color-picker">
        <color-picker class="picker" :value="colors" @input="updateValue"></color-picker>
        <div class="livestream">
          <iframe
            width="560"
            height="315"
            :src="livestream"
            frameborder="0"
            allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
            allowfullscreen
          ></iframe>
        </div>
      </div>
      <p :style="{ 'color': inverse }">
        <b>{{ message }}</b>
      </p>
    </div>
    <footer>
      <p :style="{ 'color': inverse }">
        This is Serverless. Built with
        <a
          href="https://code.visualstudio.com/tutorials/functions-extension/getting-started?WT.mc_id=ienjoylamp-dotcom-buhollan"
        >Azure Functions.</a>
      </p>
      <p class="animate" :style="{ 'color': inverse }">
        Use ENJOYLAMP20 for a 20% discount on your own
        <a
          href="https://www.lifx.com/products/lifx-mini-color-e26"
        >LIFX smart bulb</a>.
      </p>
    </footer>
  </div>
</template>

<script>
import { Sketch } from "vue-color";
import * as axios from "axios";
import { Spinner } from "spin.js";
import ColorInverter from "./inverter.js";
import * as signalR from "@aspnet/signalr";

const apiBaseUrl =
  process.env.API_BASE_URL || "https://i-enjoy-lamp.azurewebsites.net";

const colors = {
  hex: "#194d33",
  hsl: { h: 150, s: 0.5, l: 0.2, a: 1 },
  hsv: { h: 150, s: 0.66, v: 0.3, a: 1 },
  rgba: { r: 25, g: 77, b: 51, a: 1 },
  a: 1
};

export default {
  name: "App",
  data() {
    return {
      colors,
      message: "",
      inverse: "#fff",
      livestream: "https://www.youtube.com/embed/TD44SkQySa8"
    };
  },
  components: {
    ColorPicker: Sketch
  },
  mounted: async function() {
    let connection = new signalR.HubConnectionBuilder()
      .withUrl(`${apiBaseUrl}/api`)
      .build();

    connection.on(
      "colorChanged",
      function(hex) {
        this.colors = { hex: `#${hex}` };
        this.inverse = `#${ColorInverter.invertHex(hex)}`;
        this.message = `The lamp is now ${this.colors.hex}`;
      }.bind(this)
    );

    await connection.start();
  },
  methods: {
    updateValue(value) {
      // strip the # off the hex
      let hex = value.hex.substring(1);

      var opts = {
        length: 38, // The length of each line
        width: 17, // The line thickness
        radius: 45, // The radius of the inner circle
        scale: 2, // Scales overall size of the spinner
        color: `#${hex}`, // CSS color or array of colors
        animation: "spinner-line-fade-quick", // The CSS animation name for the lines
        direction: 1, // 1: clockwise, -1: counterclockwise
        className: "spinner", // The CSS class to assign to the spinner
        top: "50%", // Top position relative to parent
        left: "50%", // Left position relative to parent
        shadow: "0 0 1px transparent", // Box-shadow for the lines
        position: "absolute" // Element positioning
      };

      let target = this.$refs.target;
      let spinner = new Spinner(opts).spin(target);

      axios.get(`${apiBaseUrl}/api/setColor?color=${hex}`).then(response => {
        spinner.stop();
      });
    }
  }
};
</script>

<style>
@import url("https://fonts.googleapis.com/css?family=Spirax");

html,
body {
  height: 100%;
  font-family: sans-serif;
  padding: 0;
  margin: 0;
}

a {
  color: inherit;
}

.main {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-content: center;
  flex-direction: column;
}

footer {
  text-align: center;
}

.color {
  margin: auto;
}

.color-picker {
  display: flex;
  justify-content: center;
}

.title {
  padding: 20px;
  text-align: center;
  color: #444;
}

h1 {
  font-family: "Spirax", cursive;
  font-size: 8em;
  margin: 0;
}

.animate {
  transition: color 1s, background-color 1s;
}

@media screen and (max-width: 819px) {
  .livestream iframe {
    display: none;
  }
}

@media screen and (min-width: 820px) {
  #livestreamLink {
    display: none;
  }
}

@keyframes spinner-line-fade-more {
  0%,
  100% {
    opacity: 0; /* minimum opacity */
  }
  1% {
    opacity: 1;
  }
}

@keyframes spinner-line-fade-quick {
  0%,
  39%,
  100% {
    opacity: 0.25; /* minimum opacity */
  }
  40% {
    opacity: 1;
  }
}

@keyframes spinner-line-fade-default {
  0%,
  100% {
    opacity: 0.22; /* minimum opacity */
  }
  1% {
    opacity: 1;
  }
}
</style>
