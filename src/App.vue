<template>
  <Navigation />
  <div class="container">
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
      />

      <ol-tile-layer ref="OpenStreetMapLayer" title="Map">
        <ol-source-osm />
      </ol-tile-layer>

      <ol-vector-layer ref="bus5LineLayer" title="Bus 5 line">
        <ol-source-vector :url="bus_5_line_url" :format="geoJson">
          <ol-style :overrideStyleFunction="bus5LineStyleFn"></ol-style>
        </ol-source-vector>
      </ol-vector-layer>

      <ol-vector-layer ref="tram46LineLayer" title="Tram 46 line">
        <ol-source-vector :url="tram_4_6_line_url" :format="geoJson">
          <ol-style :overrideStyleFunction="tram46LineStyleFn"></ol-style>
        </ol-source-vector>
      </ol-vector-layer>

      <ol-vector-layer ref="tram46PointsLayer" title="Tram 46 stations">
        <ol-source-vector :url="tram_4_6_points_url" :format="geoJson">
          <ol-style :overrideStyleFunction="tram46PointsStyleFn"></ol-style>
        </ol-source-vector>
      </ol-vector-layer>

      <ol-vector-layer
        ref="PassengerNumberCircle"
        title="PassengerNumberCircle"
        v-for="passengerNumberData in passengerNumberDataList"
        :key="passengerNumberData.idx"
      >
        <ol-source-vector>
          <ol-feature>
            <ol-geom-circle
              :center="passengerNumberData.coordinates"
              :radius="passengerNumberData.numPassengers / 200000"
            ></ol-geom-circle>
            <ol-style>
              <ol-style-stroke color="blue" :width="1"></ol-style-stroke>
              <ol-style-fill color="rgba(255,200,0,0.2)"></ol-style-fill>
            </ol-style>
          </ol-feature>
        </ol-source-vector>
      </ol-vector-layer>

      <ol-rotate-control></ol-rotate-control>
      <ol-interaction-link />
      <ol-layerswitcher-control
        v-if="layerList.length > 0"
        :displayInLayerSwitcher="checkingSomething"
      />
    </ol-map>

    <input type="time" @change="updatePassengerNumbersByTime" />
  </div>
</template>

<script setup>
import { ref, inject, onMounted } from 'vue'
import Navigation from './components/Navigation.vue'
import { Fill, Style, Stroke, Circle, Text } from 'ol/style'
import { tram46_stations_passenger_number } from './data/tram_4_6_stations_passenger_number.js'

function updatePassengerNumbersByTime(e) {
  const givenTime = e.target.value
  const newDataList = tram46_stations_passenger_number.features
    .map((d, idx) => {
      if (d.geometry) {
        const coordinates = d.geometry.coordinates
        const numPassengers = d.properties[givenTime]
        if (numPassengers != null) {
          return {
            coordinates: coordinates,
            numPassengers: numPassengers,
            idx: idx
          }
        }
      }
    })
    .filter((d) => d)
  passengerNumberDataList.value = newDataList
}

const passengerNumberDataList = ref([])

const center = ref([19.048293905125572, 47.493834801228868]) // Budapest
const zoom = ref(14)
const rotation = ref(0)

const projectionName = 'EPSG:4326'

// Geojson data niihaa projectoiniig taaruulah heregtei. Taaruulaagui baigaa bolohoor tseg bolj haragdaad baina.
// const projectionDef =
//   '+proj=somerc +lat_0=47.1443937222222 +lon_0=19.0485717777778 +k_0=0.99993 +x_0=650000 +y_0=200000 +ellps=GRS67 +units=m +no_defs'
// const projectionExtent = [16.11, 45.74, 22.9, 48.58]

const format = inject('ol-format')
const geoJson = new format.GeoJSON()

const bus_5_line_url = ref('../data/bus_5_line.geojson')
const tram_4_6_line_url = ref('../data/tram_4_6_line.geojson')
const tram_4_6_points_url = ref('../data/tram_4_6_points.geojson')

function checkingSomething(input) {
  if (input.values_.title == 'PassengerNumberCircle') {
    return false
  }
  return true
}

function tram46PointsStyleFn(feature) {
  return new Style({
    image: new Circle({
      radius: 4,
      fill: new Fill({ color: 'red' }),
      stroke: new Stroke({
        color: '#00000',
        width: 1
      })
    }),
    text: new Text({
      text: feature.get('name') + '',
      textAlign: 'left',
      offsetX: 13,
      font: '12px sans-serif',
      stroke: new Stroke({ width: 4, color: 'white' })
    })
  })
}

function bus5LineStyleFn(feature) {
  return new Style({
    stroke: new Stroke({
      color: '#1861d6',
      width: 3
    })
  })
}

function tram46LineStyleFn(feature) {
  return new Style({
    stroke: new Stroke({
      color: '#f09e24',
      width: 3
    })
  })
}

const layerList = ref([])

const OpenStreetMapLayer = ref(null)
const bus5LineLayer = ref(null)
const tram46LineLayer = ref(null)
const tram46PointsLayer = ref(null)
const PassengerNumberCircle = ref(null)

onMounted(() => {
  layerList.value.push(OpenStreetMapLayer.value.tileLayer)
  layerList.value.push(bus5LineLayer.value.tileLayer)
  layerList.value.push(tram46LineLayer.value.tileLayer)
  layerList.value.push(tram46PointsLayer.value.tileLayer)
  if (PassengerNumberCircle.value) {
    layerList.value.push(PassengerNumberCircle.value.tileLayer)
  }
})
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