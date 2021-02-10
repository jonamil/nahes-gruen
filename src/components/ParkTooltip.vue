<template>
  <div :class="'tooltip ' + pickedPark.tooltipOrientation">
    <div class="inner">
      <h2>{{ parkName }}</h2>
      <span>{{ parkProperties.ortsteil }}</span>
      <div class="property size">
        <h3>Fläche</h3>
        <span>{{ parkSize }}</span>
      </div>
      <div :class="'property noise' + (controlState.parkProperty === 'noise' ? ' active' : '')">
        <h3>Verkehrslärm</h3>
        <div class="rating">
          <div :class="parkProperties.noise >= ratingThresholds.noise[0] ? 'filled' : ''"></div>
          <div :class="parkProperties.noise >= ratingThresholds.noise[1] ? 'filled' : ''"></div>
          <div :class="parkProperties.noise >= ratingThresholds.noise[2] ? 'filled' : ''"></div>
          <div :class="parkProperties.noise >= ratingThresholds.noise[3] ? 'filled' : ''"></div>
          <div :class="parkProperties.noise >= ratingThresholds.noise[4] ? 'filled' : ''"></div>
        </div>
        <span>{{ parkNoise }}</span>
      </div>
      <div :class="'property vegetation' + (controlState.parkProperty === 'vegetation' ? ' active' : '')">
        <h3>Vegetation</h3>
        <div class="rating">
          <div :class="parkProperties.veg > ratingThresholds.vegetation[0] ? 'filled' : ''"></div>
          <div :class="parkProperties.veg > ratingThresholds.vegetation[1] ? 'filled' : ''"></div>
          <div :class="parkProperties.veg > ratingThresholds.vegetation[2] ? 'filled' : ''"></div>
          <div :class="parkProperties.veg > ratingThresholds.vegetation[3] ? 'filled' : ''"></div>
          <div :class="parkProperties.veg > ratingThresholds.vegetation[4] ? 'filled' : ''"></div>
        </div>
        <span>{{ parkVegetation }}</span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ParkTooltip',

  props: {
    pickedPark: {
      type: Object,
      default: () => {
        return {
          feature: false
        }
      }
    },
    controlState: {
      type: Object,
      default: () => {
        return {
          parkProperty: 'none'
        }
      }
    }
  },

  data () {
    return {
      ratingThresholds: {
        noise: [0, 50, 55, 60, 65],
        vegetation: [0, 3.4, 6.7, 10.7, 15.5]
      }
    }
  },

  computed: {
    parkProperties() {
      if (this.pickedPark.feature === false) {
        return {
          name: '',
          ortsteil: '',
          size: 0,
          noise: 0,
          veg: 0,
          benches: 0,
          toilets: 0,
          playgrounds: 0
        }
      } else {
        return this.pickedPark.feature.properties;
      }
    },
    parkName() {
      if (this.parkProperties.name !== null) {
        return this.parkProperties.name;
      } else {
        return 'Waldgebiet';
      }
    },
    parkSize() {
      return new Intl.NumberFormat('de-DE').format(this.parkProperties.size.toFixed(1)) + ' ha';
    },
    parkNoise() {
      return new Intl.NumberFormat('de-DE').format(this.parkProperties.noise) + ' db';
    },
    parkVegetation() {
      if (this.parkProperties.veg !== null) {
        return new Intl.NumberFormat('de-DE').format(this.parkProperties.veg.toFixed(1)) + ' m³/m²';
      } else {
        return 'k.A.';
      }
    }
  }
}
</script>

<style>
.tooltip {
  position: relative;
  width: 224rem;
  height: 250rem;
  pointer-events: none;
  z-index: 50;
  /*background: #f00;*/
}

.tooltip .inner {
  position: absolute;
  box-sizing: border-box;
  bottom: 0;
  width: 100%;
  /*min-height: 200rem;*/
  padding: 16rem 20rem 17rem;
  border-radius: 25rem;
  background: #FCFBF7;
  box-shadow: 0rem 8rem 8rem rgba(0,0,0,0.03),
              0rem 4rem 4rem rgba(0,0,0,0.03),
              0rem 2rem 2rem rgba(0,0,0,0.03);
}

.tooltip.below .inner {
  top: 0;
  bottom: auto;
}

.tooltip h2 {
  margin: 0;
  text-align: center;
  font-size: 14rem;
  font-weight: 700;
}

.tooltip h2 + span, .tooltip h3 {
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5rem;
}

.tooltip h2 + span {
  display: block;
  margin: 5rem 0 19rem;
  text-align: center;
  font-size: 11rem;
  color: rgba(0,0,0,0.3);
}

.tooltip .property {
  margin-top: 14rem;
}

.tooltip .property:after {
  display: table;
  content: '';
  clear: both;
}

.tooltip .property.size {
  margin-bottom: 15rem;
}

.tooltip .property h3 {
  margin: 0 0 1rem;
  font-size: 12rem;
  color: rgba(0,0,0,0.65);
}

.tooltip .property.noise.active h3 {
  color: #E0580C;
}

.tooltip .property.vegetation.active h3 {
  color: #29955A;
}

.tooltip .property span {
  font-size: 14rem;
  font-weight: 600;
}

.tooltip .property .rating {
  float: left;
  margin-top: 4rem;
}

.tooltip .property .rating div {
  display: inline-block;
  width: 14rem;
  height: 14rem;
  margin-right: 6rem;
  border-radius: 50%;
  box-shadow: inset 0 0 0 1.5rem rgba(0,0,0,0.15);
}

.tooltip .property .rating div.filled {
  background: rgba(102,102,102,0.44);
  box-shadow: none !important;
}

.tooltip .property .rating div:nth-child(2).filled {
  background: rgba(102,102,102,0.58);
}

.tooltip .property .rating div:nth-child(3).filled {
  background: rgba(102,102,102,0.72);
}

.tooltip .property .rating div:nth-child(4).filled {
  background: rgba(102,102,102,0.86);
}

.tooltip .property .rating div:nth-child(5).filled {
  background: rgba(102,102,102,1);
}

.tooltip .property .rating + span {
  float: right;
  display: inline-block;
  margin-top: 4rem;
}

/* Custom noise rating styles */

.tooltip .property.noise.active .rating div {
  box-shadow: inset 0 0 0 1.5rem rgba(224,88,12,0.2); 
}

.tooltip .property.noise.active .rating div.filled {
  background: #F7D59A;
}

.tooltip .property.noise.active .rating div:nth-child(2).filled {
  background: #F7C27C;
}

.tooltip .property.noise.active .rating div:nth-child(3).filled {
  background: #F4A24F;
}

.tooltip .property.noise.active .rating div:nth-child(4).filled {
  background: #EB802E;
}

.tooltip .property.noise.active .rating div:nth-child(5).filled {
  background: #E0580C;
}

.tooltip .property.noise.active .rating + span {
  color: #E0580C;
}

/* Custom vegetation rating styles */

.tooltip .property.vegetation.active .rating div {
  box-shadow: inset 0 0 0 1.5rem rgba(41,149,90,0.25);
}

.tooltip .property.vegetation.active .rating div.filled {
  background: #29955A;
}

.tooltip .property.vegetation.active .rating + span {
  color: #29955A;
}
</style>