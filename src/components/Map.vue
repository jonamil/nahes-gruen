<template>
  <MglMap 
    class="map"
    :accessToken="accessToken"
    :mapStyle="mapStyle"
    :center.sync="center"
    :zoom.sync="zoom"
    :minZoom="minZoom"
    :maxZoom="maxZoom"
    logo-position="bottom-left"
    @load="mapLoaded"
  >
    <MglGeojsonLayer
      sourceId="cells"
      :source="cellsSource"
      layerId="cells"
      :layer="cellsLayer"
    />
    <MglGeojsonLayer
      sourceId="parks"
      :source="parksSource"
      layerId="parks"
      :layer="parksLayer"
    />
    <MglNavigationControl position="bottom-left" />
  </MglMap>
</template>

<script>
import Mapbox from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';

import {
 MglMap,
 MglGeojsonLayer,
 MglNavigationControl
} from 'vue-mapbox';

export default {
  name: 'Map',

  components: {
    MglMap,
    MglGeojsonLayer,
    MglNavigationControl
  },

  props: {
    // Imported GeoJson content
    cells: {
      type: Object,
      default: () => {}
    },
    parks: {
      type: Object,
      default: () => {}
    },

    // Initial center and zoom values
    initialCenter: {
      type: Array,
      default: () => [13.388707, 52.517598]
    },
    initialZoom: {
      type: Number,
      default: () => 11
    },

    // Map configuration manipulated through the interface
    config: {
      type: Object,
      default: () => {
        return {
          transportMode: 'walking',
          transportDuration: 10,
          transportTicketType: 'short',
          qualityProperty: 'size'
        }
      }
    }
  },

  data () {
    return {
      // Mapbox token and style
      accessToken: 'pk.eyJ1Ijoiam9uYW1pbCIsImEiOiJja2plbTdqdXgwN2VhMnFvN3A5cGhoZjE0In0._TRwzZ4Zu9UA5B9mT9zwsg',
      mapStyle: 'mapbox://styles/jonamil/ckjemog3e3qd219vwk883dhkx',

      // Synchronized map state values
      center: null,
      zoom: null,

      // Map options
      minZoom: 9,
      maxZoom: 18,

      // Cells: source and layer objects
      cellsSource: {
        type: 'geojson',
        data: this.cells
      },
      cellsLayer: {
        type: 'fill',
        paint: {
          'fill-color': '#ff0000',
          'fill-opacity': 0.3,
          'fill-outline-color': 'transparent'
        },
        filter: ['<=', 'API Testzellen Walking_duration', this.config.transportDuration * 60]
      },

      // Parks: source and layer objects
      parksSource: {
        type: 'geojson',
        data: this.parks
      },
      parksLayer: {
        type: 'fill',
        paint: {
          'fill-color': '#00ff00',
          'fill-outline-color': 'transparent'
        }
      }
    }
  },

  computed: {
    transportDuration() {
      return this.config.transportDuration;
    }
  },

  methods: {
    mapLoaded(event) {
      this.map = event.map;
      this.prepareParkPopup();
    },
    prepareParkPopup() {
      this.parkPopup = new this.mapbox.Popup({
        closeButton: false,
        closeOnClick: false
      });

      const vm = this;

      this.map.on('mouseenter', 'parks', function(e) {
        vm.map.getCanvas().style.cursor = 'pointer';
 
        const coordinates = e.lngLat;
        const content = e.features[0].properties.NAMENR;
         
        vm.parkPopup.setLngLat(coordinates).setHTML(content).addTo(vm.map);
      });

      this.map.on('mouseleave', 'parks', function() {
        vm.map.getCanvas().style.cursor = '';
        vm.parkPopup.remove();
      });
    }
  },

  watch: {
    transportDuration: function(newDuration) {
      this.cellsLayer.filter = ['<=', 'API Testzellen Walking_duration', newDuration * 60];
    }
  },

  created () {
    // When component instance is created, set instance’s center and zoom values to those passed in as props
    this.center = this.initialCenter;
    this.zoom = this.initialZoom;

    this.mapbox = Mapbox;
    this.map = null;
  }
}
</script>

<style>
.map {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  width: 100%;
  z-index: -1;
}

.map canvas:focus {
  outline: none;
}
</style>