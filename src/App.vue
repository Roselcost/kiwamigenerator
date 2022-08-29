<template>
  <div id="app">
    <Main/>
  </div>
</template>

<script>
import Main from './components/main.vue'

export default {
  name: 'App',
  components: {
    Main
  }
}
</script>

<style>
@font-face {
  font-family: "Edo";
  src: local("Edo"), url('assets/fonts/edosz.ttf') format("truetype");
}

body {
  background-color: #2b0a0a;
  background-image: url('assets/line.png');
}

#app {
  font-family: "Myriad Pro Light", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: white;
}

/* Slider styles */

.vue-slider .vue-slider-dot-handle, .vue-slider:hover .vue-slider-dot-handle {
  border: 2px solid #a83e3e;
}
.vue-slider:hover .vue-slider-dot-handle:hover{
  border: 2px solid #c60000;
}
.vue-slider-process{
  background-color: #9b4343;
}
.vue-slider:hover .vue-slider-process{
  background-color: #cc3c3c;
}
</style>
