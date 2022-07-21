<template>
  <!-- source -->
  <figure>
    <img ref="source" src="@/assets/frog.jpeg" @load="init" />
    <figcaption>Source Image</figcaption>
  </figure>
  <figure>
    <!-- canvas -->
    <canvas ref="destination"></canvas>
    <figcaption>Dithered Canvas</figcaption>
  </figure>
</template>

<script setup>
import RgbQuant from "rgbquant";
import { ref } from "vue";
let source = ref(null);
let destination = ref(null);

var opts = {
  colors: 5, // desired palette size
  method: 2, // histogram method, 2: min-population threshold within subregions; 1: global top-population
  boxSize: [64, 64], // subregion dims (if method = 2)
  boxPxls: 2, // min-population threshold (if method = 2)
  initColors: 4096, // # of top-occurring colors  to start with (if method = 1)
  minHueCols: 0, // # of colors per hue group to evaluate regardless of counts, to retain low-count hues
  dithKern: null, // dithering kernel name, see available kernels in docs below
  dithDelta: 0, // dithering threshhold (0-1) e.g: 0.05 will not dither colors with <= 5% difference
  dithSerp: false, // enable serpentine pattern dithering
  palette: [], // a predefined palette to start with in r,g,b tuple format: [[r,g,b],[r,g,b]...]
  reIndex: false, // affects predefined palettes only. if true, allows compacting of sparsed palette once target palette size is reached. also enables palette sorting.
  useCache: true, // enables caching for perf usually, but can reduce perf in some cases, like pre-def palettes
  cacheFreq: 10, // min color occurance count needed to qualify for caching
  colorDist: "euclidean", // method used to determine color distance, can also be "manhattan"
};

const init = () => {
  // set canvas to image size
  destination.value.style.width = source.value.width + "px";
  destination.value.style.height = source.value.height + "px";
  let context = destination.value.getContext("2d");
  context.drawImage(
    source.value,
    0,
    0,
    destination.value.width,
    destination.value.height
  );
  // initiate quant instance with options
  const q = new RgbQuant(opts);
  q.sample(source.value);
  const result = q.reduce(context);
  const imgData = context.getImageData(
    0,
    0,
    destination.value.width,
    destination.value.height
  );
  imgData.data.set(result);
  context.putImageData(imgData, 0, 0);
  console.log(imgData);
};
</script>

<style>
* {
  box-sizing: border-box;
}
html,
body {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
}
body {
  padding: 1rem;
}
canvas {
  border: 1px solid black;
}
</style>
