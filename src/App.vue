<template>
  <div id="app">
    <Map
      :cells="cells"
      :parks="parks"

      :initialCenter="mapCenter"
      :initialZoom="mapZoom"

      :transportType="transportType"
      :transportMaxDuration="transportMaxDuration"
      :transportShortDistance="transportShortDistance"
      :parkVisibleProperty="parkVisibleProperty"
    />
    <MapConfiguration
      :transportType="transportType"
      :transportMaxDuration="transportMaxDuration"
      :transportShortDistance="transportShortDistance"
      :parkVisibleProperty="parkVisibleProperty"
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
      mapCenter: [52.517598, 13.388707],
      mapZoom: 12,

      // User-modifiable map settings
      transportType: 'walk',
      transportMaxDuration: 5,
      transportShortDistance: true,
      parkVisibleProperty: 'size'
    }
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
