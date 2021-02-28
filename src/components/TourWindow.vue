<template>
  <div ref="tour" :class="'tour' + (this.tourIndex !== false ? ' visible' : '')">
    <h3>Station {{ mostRecentTourIndex + 1 }} von {{ finalTourStopIndex + 1 }}</h3>
    <h1>{{ mostRecentTourStop.title }}</h1>
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
        :class="'exit' + (mostRecentTourIndex === tourStops.length - 1 ? ' final' : '')"
        @click="exitTour"
      >
        {{ mostRecentTourIndex === tourStops.length - 1 ? 'Rundgang abschließen' : 'Rundgang verlassen' }}
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
  },

  mounted () {
    this.$refs.tour.addEventListener('click', event => {
      if (event.target.localName === 'sup') {
        this.$emit('update:currentModal', 'sources');
      }
    });
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
  max-width: 54.0rem;
  min-height: 10.0rem;
  margin-bottom: 2.0rem;
  padding: 2.0rem;
  border-radius: 2.5rem;
  background: #FAF6F0;
  box-shadow: 0 0.8rem 0.8rem rgba(0,0,0,0.03),
              0 0.4rem 0.4rem rgba(0,0,0,0.03),
              0 0.2rem 0.2rem rgba(0,0,0,0.03);
  transition: visibility 0.5s ease-in-out, opacity 0.5s ease-in-out;
}

.tour.visible {
  visibility: visible;
  opacity: 1;
}

.tour h3 {
  margin: 0;
  font-size: 1.2rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05rem;
  color: rgba(0,0,0,0.5);
}

.tour h1 {
  margin: 0.2rem 0 1.7rem;
  font-size: 1.4rem;
}

.tour p {
  margin: 1.6rem 0 0;
  text-align: left;
  font-size: 1.2rem;
  line-height: 1.45;
}

.tour .navigation {
  position: relative;
  margin-top: 1.9rem;
}

.tour .navigation button {
  padding: 0 1.6rem;
  font-size: 1.2rem;
  font-weight: 600;
  line-height: 3.0rem;
  border: 0.2rem solid rgba(0,0,0,0.1);
  border-radius: 1.7rem;
  color: rgba(0,0,0,0.65);
  background: rgba(0,0,0,0.06);
}

.tour .navigation button.skip {
  position: absolute;
  width: 3.4rem;
  height: 3.4rem;
  padding: 0;
}

.tour .navigation button.skip.previous {
  left: -1px;
  background: url('../assets/icons/tour-previous.svg') center no-repeat, rgba(0,0,0,0.06);
}

.tour .navigation button.skip.next, .tour .navigation button.exit.final {
  border: 0.2rem solid rgba(0,0,0,0.22);
}

.tour .navigation button.skip.next {
  right: -1px;
  background: url('../assets/icons/tour-next.svg') center no-repeat, linear-gradient(to bottom right, rgba(255, 255, 255, 0.1) 0%, rgba(255, 255, 255, 0) 100%), #7D7D7D;
}

.tour .navigation button.exit.final {
  color: #fff;
  background: linear-gradient(to bottom right, rgba(255, 255, 255, 0.1) 0%, rgba(255, 255, 255, 0) 100%), #7D7D7D; 
}
</style>