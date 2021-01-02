<template>
  <div class="configuration">
    <section class="left">
      <div class="group reachability">
        <h3>Reachability</h3>

        <div class="field">
          <label for="transport-mode">Mode of Transport</label>
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
          <label for="transport-duration">
            {{ availableTransportModes[transportMode] }} Duration
          </label>
          <input
            id="transport-duration"
            type="range"
            min="1"
            max="30"
            step="1"
            :value="transportDuration"
            @input="$emit('update:transportDuration', parseInt($event.target.value))"
          >
          <!-- @input="updateSliderTransportDuration(parseInt($event.target.value))" -->
          {{ transportDuration }} min
        </div>

        <div v-if="transportMode === 'vbb'" class="field">
          <label for="transport-ticket">Type of Ticket</label>
          <select
            id="transport-ticket"
            :value="transportTicketType"
            @change="$emit('update:transportTicketType', $event.target.value)"
          >
            <option
              v-for="(typeTitle, typeKey) in availableTransportTicketTypes"
              :key="typeKey"
              :value="typeKey"
            >
              {{ typeTitle }}
            </option>
          </select>
        </div>

      </div>
    </section>
    <section class="right">
      <div class="group quality">
        <h3>Quality</h3>

        <div class="field">
          <label for="quality-property">Highlighted Property</label>
          <select
            id="quality-property"
            :value="qualityProperty"
            @change="$emit('update:qualityProperty', $event.target.value)"
          >
            <option
              v-for="(propertyAttributes, propertyKey) in availableQualityProperties"
              :key="propertyKey"
              :value="propertyKey"
            >
              {{ propertyAttributes.title }}
            </option>
          </select>
        </div>

      </div>
      <div v-if="topParksByQualityProperty" class="group park-ranking">
        <h3 @click="reverseParkRanking = !reverseParkRanking">
          {{ reverseParkRanking ? 'Lowest' : 'Highest' }}-Ranking Spaces
        </h3>

        <ul>
          <li v-for="(feature, index) in topParksByQualityProperty" :key="index">
            {{ feature.properties.NAMENR }}
            <span>
              {{ formatNumber(feature.properties[qualityPropertyAttributes.propertyNameInData]) }}
              <span v-html="qualityPropertyAttributes.unit" />
            </span>
          </li>
        </ul>

      </div>
    </section>
  </div>
</template>

<script>
export default {
  name: 'MapConfiguration',

  props: {
    parks: {
      type: Object,
      default: () => {}
    },
    transportMode: {
      type: String,
      default: () => 'walking'
    },
    transportDuration: {
      type: Number,
      default: () => 10
    },
    transportTicketType: {
      type: String,
      default: () => 'short'
    },
    qualityProperty: {
      type: String,
      default: () => 'size'
    }
  },

  data () {
    return {
      availableTransportModes: {
        walking: 'Walking',
        cycling: 'Cycling',
        vbb: 'Public Transport'
      },
      availableTransportTicketTypes: {
        short: 'Short Distance (1,90€)',
        any: 'No Limitation'
      },
      availableQualityProperties: {
        size: {
          title: 'Area Size',
          propertyNameInData: 'KATASTERFL',
          unit: 'm<sup>2</sup>'
        },
        noise: {
          title: 'Average Noise Level'
        },
        trees: {
          title: 'Tree Coverage'
        },
        rating: {
          title: 'Google Maps Rating'
        }
      },

      // sliderTransportDuration: undefined,
      reverseParkRanking: false
    }
  },

  computed: {
    qualityPropertyAttributes() {
      return this.availableQualityProperties[this.qualityProperty];
    },
    topParksByQualityProperty() {
      const propertyName = this.qualityPropertyAttributes.propertyNameInData;

      if (propertyName) {
        const parkFeatures = this.parks.features;

        if (this.reverseParkRanking) {
          parkFeatures.sort((a, b) => (a.properties[propertyName] > b.properties[propertyName]) ? 1 : -1);
        } else {
          parkFeatures.sort((a, b) => (a.properties[propertyName] < b.properties[propertyName]) ? 1 : -1);
        }
        
        return parkFeatures.slice(0,5);
      } else {
        return false;
      }
    }
  },

  methods: {
    // updateSliderTransportDuration(newDuration) {
    //   this.sliderTransportDuration = newDuration;
    // },
    formatNumber: function(number) {
      if (number) {
        return new Intl.NumberFormat('en-US', { maximumSignificantDigits: 6 }).format(number.toFixed(0));
      } else {
        return undefined;
      }
    }
  },

  created () {
    // this.sliderTransportDuration = this.transportDuration;
  }
}
</script>

<style>
.configuration {
  z-index: 0;
}

.configuration section {
  position: absolute;
  top: 0;
  width: 280px;
}

.configuration section.left {
  left: 12px;
}

.configuration section.right {
  right: 12px;
}

.configuration .group {
  margin-top: 12px;
  padding: 10px 12px 14px;
  border-radius: 3px;
  box-sizing: border-box;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
  background: #fff;
}

.configuration .group h3 {
  margin: 0;
  font-size: 16px;
}

.configuration .group .field {
  margin-top: 14px;
}

.configuration .group .field label {
  display: block;
  margin-bottom: 6px;
}

.configuration .group .field select, .configuration .group .field input {
  width: 100%;
}

.configuration .group ul {
  margin: 14px 0 0;
  padding: 0;
  list-style: none;
}

.configuration .group ul li:after {
  content: '';
  display: table;
  clear: both;
}

.configuration .group ul li > span {
  float: right;
  font-weight: 700;
}
</style>