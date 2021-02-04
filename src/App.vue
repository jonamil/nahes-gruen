<template>
  <div id="app" :class="controlState.introScreen ? 'intro' : ''">
    <MapView
      :cells="cells"
      :isobands="isobands"
      :parks="parks"
      :water="water"
      :tourStops="tourStops"
      :controlState="controlState"
      :mapState="mapState"
    />
    <DistrictView
      :class="controlState.contentView !== 'district' ? 'hidden' : ''"
    />
    <Controls
      :tourStops="tourStops"
      v-bind.sync="controlState"
    />
  </div>
</template>

<script>
// Main components
import MapView from './components/MapView.vue';
import DistrictView from './components/DistrictView.vue';
import Controls from './components/Controls.vue';

// JSON data
import cells from './data/cells.json';
import isobands from './data/isobands.json';
import parks from './data/parks.json';
import water from './data/water.json';
import tourStops from './data/tour-stops.json';

export default {
  name: 'App',

  components: {
    MapView,
    DistrictView,
    Controls
  },

  data () {
    return {
      // Imported JSON data
      cells,
      isobands,
      parks,
      water,
      tourStops,

      // Control state whose properties are synced with the Controls component
      controlState: {
        // Currently visible content view
        contentView: 'map',
        // Whether the intro screen is visible inside the Controls component
        introScreen: false,
        // Index of the current tour stop (if the tour is not running, this is false)
        tourIndex: false,
        // Currently selected transport mode
        transportMode: 'walking',
        // Currently selected transport duration in minutes
        transportMinutes: 5,
        // Currently selected park property
        parkProperty: 'none'
      },

      // Initial view state of the map
      mapState: {
        longitude: 13.388707,
        latitude: 52.517598,
        zoom: 11,
        pitch: 0,
        bearing: 0,
        minZoom: 10,
        maxZoom: 16
      }
    }
  },

  created () {
    document.title = 'Nahes Grün, Fernes Grün';
  }
}
</script>

<style>
@import './assets/normalize.css';
@import './assets/fonts.css';

html {
  font-size: 6.25%;
}

#app {
  width: 100%;
  height: 100%;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen-Sans, Ubuntu, Cantarell, "Helvetica Neue", sans-serif;
  font-size: 16rem;
  font-weight: 500;
  line-height: 1.25;
  color: rgba(0,0,0,0.8);
  background: #EBE3DB;
}

.content-view {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 250rem;
  right: 0;
  transition: visibility 0.1s ease-in-out, opacity 0.1s ease-in-out;
}

.content-view.hidden {
  visibility: hidden;
  opacity: 0;
}
</style>
