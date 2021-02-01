<template>
  <div :class="'tooltip ' + pickedPark.tooltipOrientation">
    <div class="inner">
      <h3>{{ parkName }}</h3>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ParkTooltip',

  props: {
    pickedPark: {
      type: Object,
      default: () => {}
    }
  },

  computed: {
    parkProperties() {
      if (this.pickedPark.feature === false) {
        return {
          name: ''
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
    }
  }
}
</script>

<style>
.tooltip {
  position: relative;
  width: 214px;
  height: 220px;
  pointer-events: none;
  z-index: 50;
}

.tooltip .inner {
  position: absolute;
  bottom: 0;
  width: 100%;
  min-height: 200px;
  box-sizing: border-box;
  padding: 16px 20px 20px;
  border-radius: 20px;
  background: #FCFBF7;
  box-shadow: 0px 8px 8px rgba(0,0,0,0.03),
              0px 4px 4px rgba(0,0,0,0.03),
              0px 2px 2px rgba(0,0,0,0.03);
}

.tooltip.below .inner {
  top: 0;
  bottom: auto;
}

.tooltip h3 {
  margin: 0;
  font-size: 14px;
  font-weight: 700;
  text-align: center;
}
</style>