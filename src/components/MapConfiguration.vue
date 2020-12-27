<template>
	<div class="configuration">
    <div class="reachability">
      <h3>Reachability</h3>
    </div>
    <div class="quality">
      <h3>Quality</h3>
    </div>
  </div>
</template>

<script>
export default {
  name: 'MapConfiguration',

  props: {
    // Still need to come up with an elegant way to communicate changes to these made through the interface back to the parent
    // (perhaps either through using an event bus and emitting events or by using Vuex)
    transportType: {
      type: String,
      default: () => 'walk'
    },
    transportMaxDuration: {
      type: Number,
      default: () => 5
    },
    transportShortDistance: {
      type: Boolean,
      default: () => true
    },
    parkVisibleProperty: {
      type: String,
      default: () => 'size'
    }
  },

  data () {
    return {
      
    }
  }
}
</script>

<style>
.configuration {
  z-index: 0;
}

.configuration > div {
  position: absolute;
  top: 12px;
  width: 280px;
  padding: 10px 12px 11px;
  border-radius: 3px;
  box-sizing: border-box;
  background: #fff;
}

.configuration .reachability {
  left: 12px;
}

.configuration .quality {
  right: 12px;
}

.configuration h3 {
  margin: 0;
  font-size: 16px;
}
</style>