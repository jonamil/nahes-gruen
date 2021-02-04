<template>
  <div class="content-view map-view">
    <div id="mapbox" ref="mapbox"></div>
    <canvas id="deck" ref="deck"></canvas>
    <ParkTooltip
      ref="tooltip"
      :pickedPark="pickedPark"
      :controlState="controlState"
      v-show="pickedPark.active"
    />
  </div>
</template>

<script>
import ParkTooltip from './ParkTooltip.vue';

import Mapbox from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';

import { Deck } from '@deck.gl/core';
import { GeoJsonLayer } from '@deck.gl/layers';
import { ContourLayer } from '@deck.gl/aggregation-layers';

export default {
  name: 'MapView',

  components: {
    ParkTooltip
  },

  props: {
    // Imported GeoJson content
    cells: {
      type: Array,
      default: () => []
    },
    isobands: {
      type: Object,
      default: () => {}
    },
    parks: {
      type: Object,
      default: () => {}
    },
    water: {
      type: Object,
      default: () => {}
    },

    // Imported array of tour stops
    tourStops: {
      type: Array,
      default: () => []
    },

    // Control state manipulated by the interface
    controlState: {
      type: Object,
      default: () => {
        return {
          contentView: 'map',
          introScreen: false,
          tourIndex: false,
          transportMode: 'walking',
          transportMinutes: 5,
          parkProperty: 'none'
        }
      }
    },

    // Initial view state of the map (assigned to the viewState variable when instance is created)
    mapState: {
      type: Object,
      default: () => {
        return {
          longitude: 0,
          latitude: 0,
          zoom: 10,
          pitch: 0,
          bearing: 0
        }
      }
    }
  },

  data () {
    return {
      // View state object for Deck.gl
      viewState: {
        longitude: null,
        latitude: null,
        zoom: null,
        minZoom: 8,
        maxZoom: 18,
        pitch: 0,
        bearing: 0
      },

      // Most recently picked park
      pickedPark: {
        active: false,
        feature: false,
        tooltipOrientation: 'above'
      },

      // Variable to store the timeout to hide the tooltip upon un-hovering a park
      tooltipTimeout: null,

      // Pixel values for tooltip placement
      tooltipDimensions: {
        width: 214,
        height: 212,
        topCursorOffset: 8,
        bottomCursorOffset: 24,
        topBoundaryDistance: 10,
        sideBoundaryDistance: 10
      },

      // For testing purposes
      isobandsLayerType: 'geojson'
    }
  },

  computed: {
    contentView() {
      return this.controlState.contentView;
    },
    transportMinutes() {
      return this.controlState.transportMinutes;
    },
    parkProperty() {
      return this.controlState.parkProperty;
    },
    waterLayer() {
      return new GeoJsonLayer({
        id: 'water',
        data: this.water,
        getLineWidth: 0,
        getFillColor: [64, 106, 160, 255]
      });
    },
    parksLayer() {
      return new GeoJsonLayer({
        id: 'parks',
        data: this.parks,
        getLineWidth: 0,
        getFillColor: park => {
          if (this.parkProperty === 'noise') {
            if      (park.properties.noise >= 65) return [224,88,12,255];
            else if (park.properties.noise >= 60) return [235,128,46,255];
            else if (park.properties.noise >= 55) return [244,162,79,255];
            else if (park.properties.noise >= 50) return [247,194,124,255];
            else                                 return [247,213,154,255];
          } else {
            return [41, 149, 90, 255];
          }
        },
        updateTriggers: {
          getFillColor: this.parkProperty
        },
        pickable: true,
        onHover: (info) => this.updateParkTooltip(info)
      })
    },
    isobandsLayer() {
      if (this.isobandsLayerType === 'contour') {
        return new ContourLayer({
          id: 'isobands',
          data: this.cells,
          cellSize: 150.5,
          aggregation: 'MAX',
          getPosition: cell => [cell.x, cell.y],
          getWeight: cell => cell.duration / (this.transportMinutes * 60) + 1,
          contours: [{ threshold: [1, 2], color: [201, 254, 83, 204] }],
          updateTriggers: {
            getWeight: this.transportMinutes
          }
        });
      } else {
        return new GeoJsonLayer({
          id: 'isobands',
          data: this.isobands,
          getLineWidth: 0,
          getFillColor: isoband => isoband.properties.index <= this.transportMinutes ? [201, 254, 83, 204] : [0, 0, 0, 0],
          updateTriggers: {
            getFillColor: this.transportMinutes
          }
        });
      }
    },
    deckLayers() {
      return [
        this.isobandsLayer,
        this.parksLayer,
        this.waterLayer
      ];
    }
  },

  methods: {
    initializeMapbox() {
      this.mapbox = new Mapbox.Map({
        // Mapbox token and style defined in .env.local file in root directory
        accessToken: process.env.VUE_APP_MAPBOX_TOKEN,
        style: process.env.VUE_APP_MAPBOX_STYLE,
        container: this.$refs.mapbox,
        interactive: false,

        center: [this.viewState.longitude, this.viewState.latitude],
        zoom: this.viewState.zoom,
        pitch: this.viewState.pitch,
        bearing: this.viewState.bearing
      });

      // Accessibility: remove Mapbox attribution links from elements accessible via tab/keyboard
      this.mapbox.on('idle', () => {
        this.$refs.mapbox.getElementsByTagName('a').forEach(function(element) {
          element.setAttribute('tabIndex', -1);
        });
      })
    },
    initializeDeck() {
      this.deck = new Deck({
        canvas: this.$refs.deck,
        controller: true,
        layers: this.deckLayers,
        
        initialViewState: this.viewState,
        onViewStateChange: ({ viewState }) => {
          if (this.mapbox !== null) {
            this.mapbox.jumpTo({
                center: [viewState.longitude, viewState.latitude],
                zoom: viewState.zoom,
                pitch: viewState.pitch,
                bearing: viewState.bearing
            });
          }
        }
      });
    },
    renderLayers(layers) {
      this.deck.setProps({
        layers: layers
      });
    },
    updateParkTooltip(info) {
      if (info.picked) {
        this.deck.setProps({ getCursor: ({ isDragging }) => isDragging ? 'grabbing' : 'pointer' });
        
        clearTimeout(this.tooltipTimeout);

        this.pickedPark.active = true;
        this.pickedPark.feature = info.object;

        let x = info.x - this.tooltipDimensions.width / 2;
        if (x < this.tooltipDimensions.sideBoundaryDistance) {
          x = this.tooltipDimensions.sideBoundaryDistance;
        } else if (x > this.deck.width - this.tooltipDimensions.width - this.tooltipDimensions.sideBoundaryDistance) {
          x = this.deck.width - this.tooltipDimensions.width - this.tooltipDimensions.sideBoundaryDistance;
        }
        
        let y = info.y - this.tooltipDimensions.height - this.tooltipDimensions.topCursorOffset;
        if (y < this.tooltipDimensions.topBoundaryDistance) {
          y = info.y + this.tooltipDimensions.bottomCursorOffset;
          this.pickedPark.tooltipOrientation = 'below';
        } else {
          this.pickedPark.tooltipOrientation = 'above';
        }

        this.$refs.tooltip.$el.style.transform = 'translate(' + x + 'rem, ' + y + 'rem)';
      } else {
        this.deck.setProps({ getCursor: ({ isDragging }) => isDragging ? 'grabbing' : 'grab' });
        
        this.tooltipTimeout = setTimeout(() => {
          this.pickedPark.active = false;
        }, 100);
      }
    }
  },

  watch: {
    // Whenever the computed deckLayers property changes, pass these new layers to Deck.gl
    deckLayers: function(layers) {
      if (this.controlState.contentView === 'map') {
        this.renderLayers(layers);
      }
    },
    // When the content view switches back to the map, pass the current deckLayers to Deck.gl
    contentView: function(contentView) {
      if (contentView === 'map') {
        this.renderLayers(this.deckLayers);
      }
    }
  },

  created () {
    // Set the viewState data variable to the initial map state passed in as a prop
    this.viewState = this.mapState;

    // Create variables for Mapbox and Deck.gl instances, which are set when component is mounted
    this.mapbox = null;
    this.deck = null;
  },

  mounted () {
    // Initialize Mapbox and Deck.gl
    this.initializeMapbox();
    this.initializeDeck();
  }
}
</script>

<style>
#mapbox, #deck {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  width: 100%;
}

#mapbox {
  font: inherit;
  background: #EBE3DB;
}

#mapbox .mapboxgl-control-container {
  z-index: 10;
}

#mapbox .mapboxgl-ctrl-attrib {
  padding: 2rem 5rem;
  font-size: 12rem;
  border-radius: 3rem 0 0 0;
  background-color: rgba(235,227,219,0.8);
}

#mapbox .mapboxgl-ctrl-attrib a {
  color: rgba(0,0,0,0.65);
}

#mapbox .mapboxgl-ctrl-attrib a:hover {
  color: inherit;
}

#mapbox .mapbox-improve-map {
  display: none;
}

.map-view canvas:focus {
  outline: none;
}
</style>