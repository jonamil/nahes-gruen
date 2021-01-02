<template>
  <MglMap
    class="map"
    :accessToken="accessToken"
    :mapStyle="mapStyle"
    :center.sync="center"
    :zoom.sync="zoom"
    :minZoom="minZoom"
    :maxZoom="maxZoom"
    :attribution-control="false"
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
      @mouseenter="showParkPopup"
      @mouseleave="hideParkPopup"
    />
    <MglPopup
      :showed="parkPopup.visible"
      :coordinates="parkPopup.coordinates"
      :closeButton="false"
      :closeOnClick="false"
      :offset="[0, -5]"
    >
      {{ parkPopupName }}
    </MglPopup>
    <MglNavigationControl position="bottom-left" />
    <MglAttributionControl position="bottom-right" :compact="false" />
  </MglMap>
</template>

<script>
import Mapbox from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';

import {
 MglMap,
 MglGeojsonLayer,
 MglPopup,
 MglNavigationControl,
 MglAttributionControl
} from 'vue-mapbox';

import centroid from '@turf/centroid';

export default {
  name: 'Map',

  components: {
    MglMap,
    MglGeojsonLayer,
    MglPopup,
    MglNavigationControl,
    MglAttributionControl
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
      },

      // Data of current park popup
      parkPopup: {
        visible: false,
        coordinates: [0, 0],
        feature: undefined
      }
    }
  },

  computed: {
    transportDuration() {
      return this.config.transportDuration;
    },
    parkPopupName() {
      if (this.parkPopup.feature) {
        return this.parkPopup.feature.properties.NAMENR;
      } else {
        return '';
      }
    }
  },

  methods: {
    mapLoaded(event) {
      this.map = event.map;
    },
    showParkPopup(event) {
      this.map.getCanvas().style.cursor = 'pointer';

      const feature = event.mapboxEvent.features[0];

      this.parkPopup.visible = true;
      this.parkPopup.coordinates = centroid(feature).geometry.coordinates;
      this.parkPopup.feature = feature;
    },
    hideParkPopup() {
      this.map.getCanvas().style.cursor = '';
      this.parkPopup.visible = false;
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

.map .mapboxgl-map {
  font: inherit;
  font-size: 14px;
}

.map .mapboxgl-ctrl-logo.mapboxgl-compact {
  width: 88px;
}

.map .mapboxgl-ctrl-attrib {
  padding: 2px 5px 3px;
  font-size: 12px;
  border-radius: 3px 0 0 0;
  background-color: rgba(255,255,255,0.65);
}

.map .mapboxgl-popup, .map .mapboxgl-popup * {
  pointer-events: none;
}

.map .mapboxgl-popup-content {
  padding: 8px 10px 9px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

.map .mapboxgl-popup-tip {
  display: none;
}
</style>