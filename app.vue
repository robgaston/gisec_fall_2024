<script setup>
import { onMounted, ref } from "vue";
import mapboxgl from "mapbox-gl";
import "mapbox-gl/dist/mapbox-gl.css";
import style from "./mapbox/style.json";
import neighborhoods from "./data/neighborhoods-with-count.json";

mapboxgl.accessToken = "pk.eyJ1Ijoicmdhc3RvbiIsImEiOiJJYTdoRWNJIn0.MN6DrT07IEKXadCU8xpUMg";

const parentContainer = ref();
const mapContainer = ref();
const attribution = ref();
const hoverFeature = ref();
const fillColorStyle = ref();

const legendEntries = computed(() => {
  let fromValue = 0;
  const entries = [];
  if (fillColorStyle.value) {
    const styleArray = fillColorStyle.value.toSpliced(0, 2);
    for (let index = 0; index < styleArray.length; index += 2) {
      const toValue = styleArray[index + 1];
      const entry = {
        color: styleArray[index],
        text: (index == styleArray.length - 1) ? `>=${fromValue}` : `${fromValue}-${toValue - 1}`
      }
      entries.push(entry);
      fromValue = toValue;
    }
  }
  return entries;
});

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
    },
    "customAttribution": attribution.value.innerHTML
  });

  map.addControl(new mapboxgl.NavigationControl());

  map.addControl(new mapboxgl.FullscreenControl({
    container: parentContainer.value
  }));

  map.on("load", () => {
    const neighborhoodsSource = map.getSource("neighborhoods");
    neighborhoodsSource.setData(neighborhoods);

    fillColorStyle.value = map.getPaintProperty("neighborhoods-fill", "fill-extrusion-color");

    map.on("mousemove", "neighborhoods-fill", (event) => {
      hoverFeature.value = event.features[0];
      map.removeFeatureState({
        source: "neighborhoods"
      });
      map.setFeatureState(event.features[0], {
        hover: true
      });
    });

    map.on("mouseleave", "neighborhoods-fill", (event) => {
      hoverFeature.value = null;
      map.removeFeatureState({
        source: "neighborhoods"
      });
    });
  });
});
</script>

<template>
  <div ref="parentContainer" class="container">
    <div class="panel">
      <div class="title">
        Cataloged Historic Sites in Los Angeles by Neighborhood
      </div>
      <div class="subtitle">
        Rob Gaston, October 2024
      </div>
    </div>
    <div class="content">
      <div ref="mapContainer" class="map"></div>
      <div class="panel popup" v-if="hoverFeature">
        <div class="title">
          {{ hoverFeature.properties.name }}
        </div>
        <div>
          {{ hoverFeature.properties.count }} historic site(s) cataloged
        </div>
      </div>
      <div class="panel legend">
        <div class="title">
          # of sites cataloged
        </div>
        <div v-for="entry in legendEntries">
          <span class="color" v-bind:style="{ backgroundColor: entry.color }"></span>
          <span>{{ entry.text }}</span>
        </div>
      </div>
    </div>
  </div>
  <div ref="attribution" class="hidden">
    <a href="https://hpla.lacity.org/" target="_blank">
      Historic Places LA
    </a>
    |
    <a href="https://geohub.lacity.org/" target="_blank">
      Los Angeles Geohub
    </a>
  </div>
</template>

<style scoped>
.map {
  height: 100%;
}

.hidden {
  display: none;
}

.container {
  display: flex;
  flex-flow: column;
  height: 100%;
}

.content {
  flex-grow: 1;
  position: relative;
}

.panel {
  padding: 10px;
  border: 1px solid black;
  background-color: rgb(190, 190, 190);
}

.popup {
  position: absolute;
  top: 10px;
  right: 50px;
  font-size: 0.8em;
}

.legend {
  position: absolute;
  bottom: 50px;
  right: 10px;
  font-size: 0.7em;
}

.color {
  display: inline-block;
  height: 10px;
  width: 10px;
  border: 1px solid black;
  margin-right: 6px;
}

.title {
  font-weight: bold;
}

.subtitle {
  font-size: 0.9em;
}
</style>