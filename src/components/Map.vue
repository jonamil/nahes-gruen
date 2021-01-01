<template>
	<LMap
    class="map"
    :center="center"
    :zoom="zoom"
    :bounds="bounds"
    :options="mapOptions"
    @update:center="centerUpdate"
    @update:zoom="zoomUpdate"
    @update:bounds="boundsUpdate"
  >
    <!-- <LTileLayer
      url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
      attribution="&copy; <a href='http://osm.org/copyright'>OpenStreetMap</a> contributors"
    /> -->
    <LTileLayer
      url="https://api.mapbox.com/styles/v1/ellaeisemann/ckj74ra891xgq19mw7779128n/tiles/256/{z}/{x}/{y}@2x?access_token=pk.eyJ1IjoiZWxsYWVpc2VtYW5uIiwiYSI6ImNrajcydTJ5ejBqZHcyd3J1bHBhb3lyY2cifQ.FZSE_jY7uZsa14NvSxnnIQ"
      attribution="&copy; <a href='http://osm.org/copyright'>OpenStreetMap</a> contributors"
    />
    <LGeoJson
      :geojson="cells"
      :options="cellOptions"
    />
    <LGeoJson
      :geojson="parks"
      :options="parkOptions"
      :options-style="parkStyle"
    />
    <LControlZoom position="bottomleft" />
  </LMap>
</template>

<script>
import 'leaflet/dist/leaflet.css';

import {
  LMap,
  LTileLayer,
  LGeoJson,
  LControlZoom,
  // LMarker,
  // LPopup,
  // LTooltip,
} from 'vue2-leaflet';

export default {
  name: 'Map',

  components: {
    LMap,
    LTileLayer,
    LGeoJson,
    LControlZoom,
    // LMarker,
    // LPopup,
    // LTooltip,
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
      default: () => [52.517598, 13.388707]
    },
    initialZoom: {
      type: Number,
      default: () => 12
    },

    // Map configuration defined through the interface
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
      // Variables storing current state of map
      center: null,
      zoom: null,
      bounds: null,

      // Leaflet map options object
      mapOptions: {
        zoomControl: false,
        zoomSnap: 0.5,
        minZoom: 11,
        maxZoom: 15
      },

      // Style options for parks layer
      parkStyle: {
        weight: 1,
        color: '#000',
        fillColor: '#00ff00',
        fillOpacity: 1
      },
    }
  },

  computed: {
    // Assign the cell and park layers functions that will be called on each of their features (through Leaflet options object)
    cellOptions() {
      return {
        onEachFeature: this.setCellStyle
      }
    },
    parkOptions() {
      return {
        onEachFeature: this.bindParkTooltip
      }
    }
  },

  methods: {
    centerUpdate(center) {
      this.center = center;
    },
    zoomUpdate(zoom) {
      this.zoom = zoom;
    },
    boundsUpdate(bounds) {
      this.bounds = bounds;
    },
    setCellStyle(feature, layer) {
      let opacity;

      // When the transportMaxDuration value gets updated after creation, this change is not being applied yet…
      // Have to find a way to re-run the onEachFeature function each time the value gets changed
      if (feature.properties['API Testzellen Walking_duration'] <= this.transportMaxDuration * 60) {
        opacity = 0.5;
      } else {
        opacity = 0.15;
      }

      layer.setStyle({
        weight: 0,
        fillColor: '#0000ff',
        fillOpacity: opacity
      });
    },
    bindParkTooltip(feature, layer) {
      layer.bindTooltip(
        feature.properties.NAMENR,
        { permanent: false, sticky: false }
      );
    }
  },

  created: function() {
    // When map instance is created, set instance’s center and zoom values to those passed in as props
    this.center = this.initialCenter;
    this.zoom = this.initialZoom;
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
</style>