<template>
  <div id="app">
    <Map
      :cells="cells"
      :parks="parks"
      :initialCenter="mapCenter"
      :initialZoom="mapZoom"
      :config="mapConfig"
    />
    <MapConfiguration
      :parks="parks"
      :transportMode.sync="mapConfig.transportMode"
      :transportDuration.sync="mapConfig.transportDuration"
      :transportTicketType.sync="mapConfig.transportTicketType"
      :qualityProperty.sync="mapConfig.qualityProperty"
    />
  </div>
</template>

<script>
import Map from './components/Map.vue';
import MapConfiguration from './components/MapConfiguration.vue';

import cells from './data/testcells.json';
import parks from './data/testparks.json';

export default {
  name: 'App',

  components: {
    Map,
    MapConfiguration
  },

  data () {
    return {
      // Imported GeoJson content
      cells,
      parks,

      // Initial location values for map
      mapCenter: [13.388707, 52.517598],
      mapZoom: 11,

      // User-modifiable map configuration
      mapConfig: {
        transportMode: 'walking',
        transportDuration: 10,
        transportTicketType: 'short',
        qualityProperty: 'size'
      }
    }
  },

  created () {
    document.title = 'Green Inclusion';
  }
}
</script>

<style>
@import './assets/normalize.css';
@import './assets/fonts.css';

#app {
  width: 100%;
  height: 100%;
  font-family: 'PlexSansWeb', 'IBM Plex Sans', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen-Sans, Ubuntu, Cantarell, "Helvetica Neue", sans-serif;
  font-size: 16px;
  line-height: 1.25;
}
</style>
