<template>
  <div :class="'tooltip ' + pickedPark.tooltipOrientation">
    <div class="inner">
      <h2>{{ parkName }}</h2>
      <span>{{ parkProperties.ortsteil }}</span>
      <div class="columns">
        <div class="property size">
          <h3>Fläche</h3>
          <span>{{ parkSize }} ha</span>
        </div>
        <div class="property amenities">
          <h3>Ausstattung</h3>
          <div :class="'icon benches' + (parkProperties.benches > 0 ? '' : ' disabled')" />
          <div :class="'icon toilets' + (parkProperties.toilets > 0 ? '' : ' disabled')" />
          <div :class="'icon playgrounds' + (parkProperties.playgrounds > 0 ? '' : ' disabled')" />
        </div>
      </div>
      <div :class="'property noise' + (controlState.parkProperty === 'noise' ? ' active' : '')">
        <h3>Lärmbelastung</h3>
        <div class="rating">
          <div :class="parkProperties.noise > ratingThresholds.noise[0] ? 'filled' : ''"></div>
          <div :class="parkProperties.noise >= ratingThresholds.noise[1] ? 'filled' : ''"></div>
          <div :class="parkProperties.noise >= ratingThresholds.noise[2] ? 'filled' : ''"></div>
          <div :class="parkProperties.noise >= ratingThresholds.noise[3] ? 'filled' : ''"></div>
          <div :class="parkProperties.noise >= ratingThresholds.noise[4] ? 'filled' : ''"></div>
        </div>
        <span>{{ parkNoise }} dB<sub>A</sub></span>
      </div>
      <div :class="'property vegetation' + (controlState.parkProperty === 'vegetation' ? ' active' : '')">
        <h3>Vegetationsmenge</h3>
        <div class="rating">
          <div :class="parkProperties.veg > ratingThresholds.vegetation[0] ? 'filled' : ''"></div>
          <div :class="parkProperties.veg >= ratingThresholds.vegetation[1] ? 'filled' : ''"></div>
          <div :class="parkProperties.veg >= ratingThresholds.vegetation[2] ? 'filled' : ''"></div>
          <div :class="parkProperties.veg >= ratingThresholds.vegetation[3] ? 'filled' : ''"></div>
          <div :class="parkProperties.veg >= ratingThresholds.vegetation[4] ? 'filled' : ''"></div>
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
        vegetation: [0, 3, 5, 10, 15]
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
      return new Intl.NumberFormat('de-DE').format(this.parkProperties.size.toFixed(1));
    },
    parkNoise() {
      return new Intl.NumberFormat('de-DE').format(this.parkProperties.noise);
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
  width: 22.4rem;
  height: 25.0rem;
  pointer-events: none;
  z-index: 50;
  /*background: #f00;*/
}

.tooltip .inner {
  position: absolute;
  box-sizing: border-box;
  bottom: 0;
  width: 100%;
  /*min-height: 20.0rem;*/
  padding: 1.6rem 2.0rem 1.7rem;
  border-radius: 2.5rem;
  background: #FCFBF7;
  box-shadow: 0 0.8rem 0.8rem rgba(0,0,0,0.03),
              0 0.4rem 0.4rem rgba(0,0,0,0.03),
              0 0.2rem 0.2rem rgba(0,0,0,0.03);
}

.tooltip.below .inner {
  top: 0;
  bottom: auto;
}

.tooltip h2 {
  margin: 0;
  text-align: center;
  font-size: 1.4rem;
  font-weight: 700;
}

.tooltip h2 + span, .tooltip h3 {
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05rem;
}

.tooltip h2 + span {
  display: block;
  margin: 0.5rem 0 0.5rem;
  text-align: center;
  font-size: 1.1rem;
  color: rgba(0,0,0,0.3);
}

.tooltip .columns {
  margin-bottom: 1.5rem;
}

.tooltip .columns:after, .tooltip .property:after {
  display: table;
  content: '';
  clear: both;
}

.tooltip .property {
  margin-top: 1.4rem;
}

.tooltip .property.size {
  float: left;
}

.tooltip .property.amenities {
  float: right;
  text-align: center;
}

.tooltip .property h3 {
  margin: 0 0 0.1rem;
  font-size: 1.2rem;
  color: rgba(0,0,0,0.65);
}

.tooltip .property span {
  font-size: 1.4rem;
  font-weight: 600;
}

.tooltip .property.amenities .icon {
  display: inline-block;
  opacity: 0.75;
  width: 2.6rem;
  height: 2.6rem;
  margin: -0.2rem 0.4rem -0.8rem;
  /*background-color: #f00;*/
  background-position: center;
  background-repeat: no-repeat;
}

.tooltip .property.amenities .icon:first-of-type {
  margin-left: -1.0rem;
}

.tooltip .property.amenities .icon:last-of-type {
  margin-right: -1.0rem;
}

.tooltip .property.amenities .icon.disabled {
  opacity: 0.18; 
}

.tooltip .property.amenities .icon.benches {
  background-image: url('../assets/icons/benches.svg');
}

.tooltip .property.amenities .icon.benches.disabled {
  background-image: url('../assets/icons/benches-disabled.svg');
}

.tooltip .property.amenities .icon.toilets {
  background-image: url('../assets/icons/toilets.svg');
}

.tooltip .property.amenities .icon.toilets.disabled {
  background-image: url('../assets/icons/toilets-disabled.svg');
}

.tooltip .property.amenities .icon.playgrounds {
  background-image: url('../assets/icons/playgrounds.svg');
}

.tooltip .property.amenities .icon.playgrounds.disabled {
  background-image: url('../assets/icons/playgrounds-disabled.svg');
}

.tooltip .property .rating {
  float: left;
  margin-top: 0.4rem;
}

.tooltip .property .rating div {
  display: inline-block;
  width: 1.4rem;
  height: 1.4rem;
  margin-right: 0.6rem;
  border-radius: 50%;
  box-shadow: inset 0 0 0 0.15rem rgba(0,0,0,0.15);
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
  margin-top: 0.4rem;
}

/* Custom noise rating styles */

.tooltip .property.noise.active h3 {
  color: #E0580C;
}

.tooltip .property.noise.active .rating div {
  box-shadow: inset 0 0 0 0.15rem rgba(224,88,12,0.2); 
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

.tooltip .property.vegetation.active h3 {
  color: #227D52;
}

.tooltip .property.vegetation.active .rating div {
  box-shadow: inset 0 0 0 0.15rem rgba(41,149,90,0.25);
}

.tooltip .property.vegetation.active .rating div.filled {
  background: #35C25D;
}

.tooltip .property.vegetation.active .rating div:nth-child(2).filled {
  background: #2FAB5C;
}

.tooltip .property.vegetation.active .rating div:nth-child(3).filled {
  background: #29955A;
}

.tooltip .property.vegetation.active .rating div:nth-child(4).filled {
  background: #227D52;
}

.tooltip .property.vegetation.active .rating div:nth-child(5).filled {
  background: #1C664B;
}

.tooltip .property.vegetation.active .rating + span {
  color: #227D52;
}
</style>