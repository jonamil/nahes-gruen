<template>
  <div class="controls">
    <div class="sidebar">
      <div class="group logo">
        <img src="../assets/logo.svg" width="170" height="81" />
      </div>
      <div class="group reachability">
        <div class="field">
          <label for="transport-mode">Fortbewegung</label>
          <select
            id="transport-mode"
            :value="transportMode"
            @change="$emit('update:transportMode', $event.target.value)"
          >
            <option
              v-for="(modeTitle, modeKey) in availableTransportModes"
              :key="modeKey"
              :value="modeKey"
            >
              {{ modeTitle }}
            </option>
          </select>
        </div>
        <div class="field">
          <label for="transport-minutes">Zeitaufwand</label>
          <input
            id="transport-minutes"
            type="range"
            min="1"
            max="20"
            step="1"
            :value="transportMinutes"
            @input="$emit('update:transportMinutes', parseInt($event.target.value))"
          >
          <!-- @input="updateSlidertransportMinutes(parseInt($event.target.value))" -->
          {{ transportMinutes }} Min.
        </div>
      </div>
      <div class="group properties">
        <div class="field">
          <label for="park-property">Merkmale</label>
          <select
            id="park-property"
            :value="parkProperty"
            @change="$emit('update:parkProperty', $event.target.value)"
          >
            <option
              v-for="(propertyAttributes, propertyKey) in availableParkProperties"
              :key="propertyKey"
              :value="propertyKey"
            >
              {{ propertyAttributes.title }}
            </option>
          </select>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Controls',

  props: {
    contentView: {
      type: String,
      default: 'map'
    },
    introScreen: {
      type: Boolean,
      default: false
    },
    tourIndex: {
      type: [Number, Boolean],
      default: false
    },
    transportMode: {
      type: String,
      default: 'walking'
    },
    transportMinutes: {
      type: Number,
      default: 5
    },
    parkProperty: {
      type: String,
      default: 'none'
    }
  },

  data () {
    return {
      availableTransportModes: {
        walking: 'Zu Fuß',
        cycling: 'Mit dem Fahrrad'
      },
      availableParkProperties: {
        none: {
          title: 'Keine Auswahl'
        },
        noise: {
          title: 'Verkehrslärm'
        },
        vegetation: {
          title: 'Vegetation'
        }
      },

      // sliderTransportMinutes: undefined
    }
  },

  computed: {
    
  },

  methods: {
    // updateSliderTransportMinutes(newDuration) {
    //   this.sliderTransportMinutes = newDuration;
    // }
  },

  created () {
    // this.sliderTransportMinutes = this.TransportMinutes;
  }
}
</script>

<style>
.controls {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 100;
  pointer-events: none;
}

.controls > * {
  pointer-events: auto;
}

.controls .sidebar {
  position: relative;
  overflow: scroll;
  box-sizing: border-box;
  width: 250px;
  height: 100%;
  /*padding-bottom: 150px;*/
  background: #FAF6F0;
  box-shadow: 8px 0px 8px rgba(0,0,0,0.03),
              4px 0px 4px rgba(0,0,0,0.03),
              2px 0px 2px rgba(0,0,0,0.03);
}

.controls .group {
  padding: 0 25px;
}

.controls .group.logo {
  margin-top: 22px;
  text-align: center;
}

.controls .group.reachability {
  margin-top: 35px;
}

.controls .group.properties {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  /*height: 150px;*/
  margin-bottom: 28px;
}

.controls .group .field {
  margin-top: 14px;
}

.controls .group .field label {
  display: block;
  margin-bottom: 6px;
}

.controls .group .field select, .controls .group .field input {
  width: 100%;
}
</style>