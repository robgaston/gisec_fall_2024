<script setup>
import { onMounted, ref } from "vue";
import mapboxgl from "mapbox-gl";
import "mapbox-gl/dist/mapbox-gl.css";
import style from "./mapbox/style.json";

mapboxgl.accessToken = "";

const mapContainer = ref();

onMounted(() => {
  new mapboxgl.Map({
    "container": mapContainer.value,
    "style": style
  });
});
</script>

<template>
  <div ref="mapContainer" class="map"></div>
</template>

<style scoped>
.map {
  height: 100%;
}
</style>