<template>
  <div :class="'tour' + (this.tourIndex !== false ? ' visible' : '')">
    <h3>Station {{ mostRecentTourIndex + 1 }} von {{ finalTourStopIndex + 1 }}</h3>
    <h1><mark>{{ mostRecentTourStop.title }}</mark></h1>
    <p
      v-for="(paragraph, index) in mostRecentTourStop.paragraphs"
      :key="index"
      v-html="paragraph"
    />
    <div class="navigation">
      <button
        v-if="mostRecentTourIndex !== 0"
        class="skip previous"
        title="Vorherige Station"
        @click="previousStop"
      />
      <button
        class="exit"
        @click="exitTour"
      >
        Rundgang verlassen
      </button>
      <button
        v-if="mostRecentTourIndex !== finalTourStopIndex"
        class="skip next"
        title="Nächste Station"
        @click="nextStop"
      />
    </div>
  </div>
</template>

<script>
export default {
  name: 'TourWindow',

  props: {
    tourStops: {
      type: Array,
      default: () => []
    },
    tourIndex: {
      type: [Number, Boolean],
      default: false
    }
  },

  data () {
    return {
      mostRecentTourIndex: 0
    }
  },

  computed: {
    finalTourStopIndex() {
      return this.tourStops.length - 1;
    },
    mostRecentTourStop() {
      if (this.mostRecentTourIndex !== false && this.mostRecentTourIndex <= this.finalTourStopIndex) {
        return this.tourStops[this.mostRecentTourIndex];
      } else {
        return this.tourStops[0];
      }
    }
  },

  methods: {
    previousStop() {
      if (this.tourIndex !== 0) {
        this.$emit('update:tourIndex', this.tourIndex - 1);
      }
    },
    nextStop() {
      if (this.tourIndex !== this.finalTourStopIndex) {
        this.$emit('update:tourIndex', this.tourIndex + 1);
      }
    },
    exitTour() {
      this.$emit('update:tourIndex', false);
    }
  },

  watch: { 
    tourIndex: function(tourIndex) {
      if (tourIndex !== false && tourIndex <= this.finalTourStopIndex) {
        this.mostRecentTourIndex = tourIndex;
      }
    }
    // tourIndex: function(tourIndex) {
    //   if (tourIndex !== false && this.tourIndex <= this.finalTourStopIndex) {
    //     this.currentTourStop = this.tourStops[tourIndex];
    //   }
    // }
  },

  created () {
    // this.currentTourStop = this.tourStops[0];
  }
}
</script>

<style>
.tour {
  visibility: hidden;
  opacity: 0;
  display: inline-block;
  vertical-align: top;
  box-sizing: border-box;
  width: 75%;
  max-width: 540rem;
  min-height: 100rem;
  margin-bottom: 20rem;
  padding: 20rem;
  border-radius: 25rem;
  background: #FAF6F0;
  box-shadow: 0rem 8rem 8rem rgba(0, 0, 0, 0.03),
              0rem 4rem 4rem rgba(0, 0, 0, 0.03),
              0rem 2rem 2rem rgba(0, 0, 0, 0.03);
  transition: visibility 0.5s ease-in-out, opacity 0.5s ease-in-out;
}

.tour.visible {
  visibility: visible;
  opacity: 1;
}

.tour h3 {
  margin: 0;
  font-size: 12rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5rem;
  color: rgba(0,0,0,0.5);
}

.tour h1 {
  margin: 5rem 0 20rem;
  font-size: 14rem;
}

.tour h1 mark {
  padding: 3rem 5rem;
  border-radius: 3rem;
  color: inherit;
  background: #C9FE53;
}

.tour p {
  margin: 16rem 0 0;
  text-align: left;
  font-size: 12rem;
  line-height: 1.4;
}

.tour .navigation {
  position: relative;
  margin-top: 18rem;
  /*background: #f00;*/
}

.tour .navigation button {
  -webkit-appearance: none;
  appearance: none;
  cursor: pointer;
  box-sizing: border-box;
  padding: 0 16rem;
  font-size: 12rem;
  font-weight: 600;
  line-height: 30rem;
  border: 2rem solid rgba(0,0,0,0.1);
  border-radius: 17rem;
  color: rgba(0,0,0,0.65);
  background: rgba(0,0,0,0.06);
}

.tour .navigation button:focus {
  outline: none;
}

.tour .navigation button:focus-visible {
  box-shadow: 0 0 0 2px #FAF6F0, 0 0 0 4px rgba(0,0,0,0.1);
}

.tour .navigation button.skip {
  position: absolute;
  width: 34rem;
  height: 34rem;
  padding: 0;
}

.tour .navigation button.skip.previous {
  left: -1px;
  background: url('../assets/icons/tour-previous.svg') center no-repeat, rgba(0,0,0,0.06);
}

.tour .navigation button.skip.next {
  right: -1px;
  border: 2rem solid rgba(0,0,0,0.22);
  color: #fff;
  background: url('../assets/icons/tour-next.svg') center no-repeat, linear-gradient(to bottom right, rgba(255, 255, 255, 0.1) 0%, rgba(255, 255, 255, 0) 100%), #7D7D7D;
}
</style>