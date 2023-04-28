<template>
  <div class="h-full w-full relative">
    <l-map
      ref="map"
      :min-zoom="8"
      :useGlobalLeaflet="false"
      v-model:zoom="zoom"
      v-model:center="center"
    >
      <l-tile-layer
        :url="`http://${tileURL}/{z}/{x}/{y}.png`"
        layer-type="base"
        name="OpenStreetMap"
      ></l-tile-layer>
      <l-marker :lat-lng="start"></l-marker>
      <l-marker :lat-lng="destination"></l-marker>
      <l-polyline :lat-lngs="routePath"></l-polyline>
    </l-map>
    <div class="absolute bottom-4 left-1/2 -translate-x-1/2 z-[9999]">
      <div
        class="absolute bottom-0 left-0 z-0 opacity-15 w-full h-full rounded-lg bg-slate-900 animate-ping"
      ></div>
      <button
        @click="getRoute"
        class="relative z-50 bg-slate-900 text-cyan-400 text-center px-2 py-1 rounded shadow-md hover:bg-cyan-600 hover:text-slate-900 transition-all"
      >
        Get Route
      </button>
    </div>
  </div>
</template>

<script>
import "leaflet/dist/leaflet.css";
import { LMap, LTileLayer, LMarker, LPolyline } from "@vue-leaflet/vue-leaflet";

export default defineNuxtComponent({
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPolyline,
  },
  computed: {
    routingUrlFull() {
      const url = `${this.routingUrl}start=${this.start[1]},${this.start[0]}&end=${this.destination[1]},${this.destination[0]}`;
      console.log(url);
      return url;
    },
  },
  data() {
    return {
      zoom: 11,
      center: [30.056340067069875, 31.274975822065922],
      start: [30.113075859168305, 31.397057212138296],
      destination: [29.981135203535274, 31.133849840173017],
      tileURL: "http://ec2-3-65-28-152.eu-central-1.compute.amazonaws.com/hot",
      routingUrl:
        "http://ec2-18-195-147-254.eu-central-1.compute.amazonaws.com:8080/ors/v2/directions/driving-car?",
      routePath: [],
    };
  },
  methods: {
    async getRoute() {
      const result = await $fetch(this.routingUrlFull, {
        method: "GET",
      });
      const steps = result.features[0].properties.segments[0].steps;
      const coords = result.features[0].geometry.coordinates;
      const latLngs = coords.map((coord) => {
        const lat = coord[1];
        coord[1] = coord[0];
        coord[0] = lat;
        return coord;
      });
      this.routePath = latLngs;
    },
  },
});
</script>
