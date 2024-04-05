<template>
  <Navigation />
  <div class="container">
    <form>
      <label for="zoom">Zoom:</label>
      <input type="number" id="zoom" v-model="zoom" />
    </form>

    <ol-map style="height: 400px">
      <!--
        Geojson data niihaa projectoiniig taaruulah heregtei. Taaruulaagui baigaa bolohoor tseg bolj haragdaad baina.
        <ol-projection-register
        :projectionName="projectionName"
        :projectionDef="projectionDef"
        :projectionExtent="projectionExtent"
      /> -->

      <ol-view
        ref="view"
        :center="center"
        :rotation="rotation"
        :zoom="zoom"
        :projection="projectionName"
        @change:center="centerChanged"
        @change:resolution="resolutionChanged"
        @change:rotation="rotationChanged"
      />

      <ol-tile-layer>
        <ol-source-osm />
      </ol-tile-layer>

      <ol-vector-layer ref="vectorSourceRef">
        <ol-source-vector :url="url" :format="geoJson">
          <ol-style :overrideStyleFunction="styleFn"></ol-style>
        </ol-source-vector>
      </ol-vector-layer>

      <ol-rotate-control></ol-rotate-control>
      <ol-interaction-link />
    </ol-map>

    <ul>
      <li>center : {{ currentCenter }}</li>
      <li>resolution : {{ currentResolution }}</li>
      <li>zoom : {{ currentZoom }}</li>
      <li>rotation : {{ currentRotation }}</li>
    </ul>
  </div>
</template>

<script setup>
import { ref, inject } from 'vue'
import Navigation from './components/Navigation.vue'
import { Fill, Style, Stroke } from 'ol/style'

const center = ref([19.048293905125572, 47.493834801228868]) // Budapest
const zoom = ref(14)
const rotation = ref(0)

const currentCenter = ref(center.value)
const currentZoom = ref(zoom.value)
const currentRotation = ref(rotation.value)
const currentResolution = ref(0)

function resolutionChanged(event) {
  currentResolution.value = event.target.getResolution()
  currentZoom.value = event.target.getZoom()
}
function centerChanged(event) {
  currentCenter.value = event.target.getCenter()
}
function rotationChanged(event) {
  currentRotation.value = event.target.getRotation()
}

const projectionName = 'EPSG:4326'

// Geojson data niihaa projectoiniig taaruulah heregtei. Taaruulaagui baigaa bolohoor tseg bolj haragdaad baina.
// const projectionDef =
//   '+proj=somerc +lat_0=47.1443937222222 +lon_0=19.0485717777778 +k_0=0.99993 +x_0=650000 +y_0=200000 +ellps=GRS67 +units=m +no_defs'
// const projectionExtent = [16.11, 45.74, 22.9, 48.58]

const format = inject('ol-format')
const geoJson = new format.GeoJSON()

const url = ref('../data/line.geojson')

function styleFn(feature) {
  return new Style({
    stroke: new Stroke({
      color: '#f09e24',
      width: 3
    })
  })
}
</script>

<style scoped>
.ol-map {
  position: relative;
}
.ol-map-loading:after {
  content: '';
  box-sizing: border-box;
  position: absolute;
  top: 50%;
  left: 50%;
  width: 80px;
  height: 80px;
  margin-top: -40px;
  margin-left: -40px;
  border-radius: 50%;
  border: 5px solid rgba(180, 180, 180, 0.6);
  border-top-color: var(--vp-c-brand-1);
  animation: spinner 0.6s linear infinite;
}

@keyframes spinner {
  to {
    transform: rotate(360deg);
  }
}
</style>