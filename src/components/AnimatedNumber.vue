<template>
  <div ref="element">
    {{ formatNumber(liveNumber) }}
  </div>
</template>

<script>
export default {
  name: 'AnimatedNumber',

  props: {
    number: {
      type: Number,
      default: 0
    },
    fractionDigits: {
      type: Number,
      default: 0
    }
  },

  data () {
    return {
      liveNumber: 0,
      duration: 500,
      interval: null
    }
  },

  methods: {
    formatNumber(number) {
      return new Intl.NumberFormat('de-DE', { minimumFractionDigits: this.fractionDigits, maximumFractionDigits: this.fractionDigits }).format(number);
    }
  },

  created() {
    this.liveNumber = this.number;
  },

  watch: {
    number: function(newNumber) {
      clearInterval(this.interval);

      const startNumber = this.liveNumber;
      const startTime = new Date().getTime();
      
      this.interval = setInterval(() => {
        const progress = Math.min((new Date().getTime() - startTime) / this.duration, 1);
        const easingProgress = 1 - Math.pow(1 - progress, 5);

        const value = Math.round(easingProgress * (newNumber - startNumber) + startNumber);
        this.liveNumber = value;

        if (value === newNumber) clearInterval(this.interval);
      }, 10);      
    }
  }
}
</script>

<style scoped>
div {
  display: inline !important;
}
</style>