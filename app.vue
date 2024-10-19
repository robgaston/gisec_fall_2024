<script setup>
import { onMounted, ref } from "vue";
import mapboxgl from "mapbox-gl";
import "mapbox-gl/dist/mapbox-gl.css";
import style from "./mapbox/style.json";
import neighborhoods from "./data/neighborhoods-with-count.json";

mapboxgl.accessToken = "pk.eyJ1Ijoicmdhc3RvbiIsImEiOiJJYTdoRWNJIn0.MN6DrT07IEKXadCU8xpUMg";

const mapContainer = ref();

onMounted(() => {
  const map = new mapboxgl.Map({
    "container": mapContainer.value,
    "style": style,
    "bounds": [
      -118.65226,
      33.70555,
      -118.1557,
      34.33305
    ],
    "fitBoundsOptions": {
      "padding": 25,
      "pitch": 45
    }
  });

  map.addControl(new mapboxgl.NavigationControl());

  map.on("load", () => {
    const neighborhoodsSource = map.getSource("neighborhoods");
    neighborhoodsSource.setData(neighborhoods);
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