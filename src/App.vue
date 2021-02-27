<template>
  <div id="app" :class="controlState.introScreen ? 'intro' : ''">
    <MapView
      :isobands="isobands"
      :parks="parks"
      :water="water"
      :tourStops="tourStops"
      :controlState="controlState"
      :initialMapState="initialMapState"
    />
    <LoadingView
      :class="loadingComplete ? 'hidden' : ''"
    />
    <DistrictView
      :class="controlState.contentView !== 'district' ? 'hidden' : ''"
      :coverage="coverage"
      :controlState="controlState"
    />
    <Controls
      :tourStops="tourStops"
      :coverage="coverage"
      v-bind.sync="controlState"
    />
  </div>
</template>

<script>
// Main components
import MapView from './components/MapView.vue';
import LoadingView from './components/LoadingView.vue';
import DistrictView from './components/DistrictView.vue';
import Controls from './components/Controls.vue';

// Tour stops and coverage percentages JSON
import tourStops from './data/tour-stops.json';
import coverage from './data/coverage.json';

export default {
  name: 'App',

  components: {
    MapView,
    LoadingView,
    DistrictView,
    Controls
  },

  data () {
    return {
      // Variables for JSON geodata, which are populated once data is fetched
      isobands: null,
      parks: null,
      water: null,

      // Statically imported data
      tourStops,
      coverage,

      // Control state whose properties are synced with the Controls component
      controlState: {
        // Currently visible content view
        contentView: 'map',
        // Whether the intro screen is visible inside the Controls component
        introScreen: true,
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
      initialMapState: {
        longitude: 13.388707,
        latitude: 52.517598,
        zoom: 12,
        pitch: 0,
        bearing: 0,
        minZoom: 10,
        maxZoom: 16
      },

      loadingComplete: false
    }
  },

  computed: {
    dataFetched() {
      if (this.isobands && this.parks && this.water) {
        return true;
      } else {
        return false;
      }
    }
  },

  methods: {
    // Once JSON source is fetched, assign its contents to a data variable with the same name
    async fetchJSON(source) {
      try {
        const response = await fetch('./geodata/' + source + '.json');
        const data = await response.json();
        this[source] = data;
      } catch {
        this.[source] = null;
      }
    }
  },

  watch: {
    dataFetched: function(fetched) {
      if (fetched) {
        setTimeout(() => this.loadingComplete = true, 2000);
      }
    }
  },

  created () {
    document.title = 'Nahes Grün, Fernes Grün';

    // Asynchronously fetch each geodata source
    ['isobands', 'parks', 'water'].forEach(source => {
      this.fetchJSON(source);
    });

    // If the intro screen was hidden during a previous visit, load straight into the regular view
    if (window.localStorage.getItem('introScreen') === 'false') {
      this.controlState.introScreen = false;
    }
  }
}
</script>

<style>
@import './assets/normalize.css';
@import './assets/fonts.css';

html {
  font-size: 62.5%;
}

#app {
  width: 100%;
  height: 100%;
  font-family: Inter, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen-Sans, Ubuntu, Cantarell, 'Helvetica Neue', sans-serif;
  font-size: 1.6rem;
  font-weight: 460;
  line-height: 1.25;
  color: rgba(0,0,0,0.8);
  background: #EBE3DB;
}

.content-view {
  position: absolute;
  overflow: hidden;
  top: 0;
  bottom: 0;
  left: 25.0rem;
  right: 0;
  transition: visibility 0.1s ease-in-out, opacity 0.1s ease-in-out;
}

.content-view.hidden {
  visibility: hidden;
  opacity: 0;
}
</style>
