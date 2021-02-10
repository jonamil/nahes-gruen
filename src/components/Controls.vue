<template>
  <div class="controls" @click.self="hideIntro">
    <div ref="sidebar" class="sidebar">
      <div class="inner inputs">
        <section class="logo">
          <img src="../assets/logo.svg" height="86" alt="Nahes Grün, Fernes Grün" />
        </section>
        <section class="reachability">
          <div class="field">
            <h3>Fortbewegung</h3>
            <button title="Erläuterung" @click="currentModal = 'transportMode'" />
            <div class="vertical-toggle">
              <label
                v-for="(modeTitle, modeKey) in availableTransportModes"
                :key="modeKey"
                :class="modeKey + (modeKey === transportMode ? ' checked' : '')"
              >
                <input
                  type="radio"
                  name="transport-mode"
                  :value="modeKey"
                  :checked="modeKey === transportMode"
                  @change="$emit('update:transportMode', $event.target.value)"
                />
                <span v-html="modeTitle" />
              </label>
            </div>
          </div>
          <div class="field">
            <h3>Zeitaufwand</h3>
            <button title="Erläuterung" @click="currentModal = 'transportMinutes'" />
            <div ref="slider" class="slider">
              <span :style="sliderProperties.minValue ? 'width: ' + sliderProgressWidth + 'rem' : ''"></span>
              <input
                id="transport-minutes"
                type="range"
                min="1"
                max="15"
                step="1"
                :value="transportMinutes"
                @input="$emit('update:transportMinutes', parseInt($event.target.value))"
              >
              <label for="transport-minutes" :style="sliderProperties.minValue ? 'left: ' + sliderLabelOffset + 'rem' : ''">
                <mark>{{ transportMinutes }} Min.</mark>
              </label>
            </div>
          </div>
          <div class="field coverage">
            <h3>Abdeckung</h3>
            <mark>75,4<span>%</span></mark>
            <span>aller Berliner*innen können unter diesen Bedingungen von zuhause einen Grünraum erreichen.</span>
          </div>
        </section>
        <section :class="'properties' + (contentView !== 'map' ? ' hidden' : '')">
          <div class="field">
            <h3>Merkmale</h3>
            <button title="Erläuterung" @click="currentModal = 'properties'" />
            <div class="vertical-toggle">
              <label
                v-for="(propertyAttributes, propertyKey) in availableParkProperties"
                :key="propertyKey"
                :class="propertyKey + (propertyKey === parkProperty ? ' checked' : '')"
              >
                <input
                  type="radio"
                  name="park-property"
                  :value="propertyKey"
                  :checked="propertyKey === parkProperty"
                  @change="$emit('update:parkProperty', $event.target.value)"
                />
                <span v-html="propertyAttributes.title" />
              </label>
            </div>
          </div>
        </section>
      </div>
      <div class="inner intro">
        <section class="welcome">
          <img src="../assets/logo.svg" height="120" alt="Nahes Grün, Fernes Grün" />
          <p v-html="introContents.welcome" />
          <button @click="hideIntroAndExitTour">Karte erkunden</button>
          <button @click="startTour">
            {{ tourIndex !== false ? 'Rundgang fortsetzen' : 'Rundgang starten' }}
          </button>
        </section>
        <section class="article">

        </section>
      </div>
      <button class="close" title="Schließen" @click="hideIntro" />
    </div>
    <div class="floating top">
      <div class="view-settings">
        <button title="Einführung" @click="showIntro" />
        <div class="view-toggle">
          <template v-for="(viewTitle, viewKey) in { map: 'Karte', district: 'Bezirke' }">
            <input
              :id="viewKey + '-view'"
              :key="viewKey + '-input'"
              type="radio"
              name="content-view"
              :value="viewKey"
              :checked="contentView === viewKey"
              @change="$emit('update:contentView', $event.target.value)"
            />
            <label
              :key="viewKey + '-label'"
              :for="viewKey + '-view'"
              v-text="viewTitle"
            />
          </template>
        </div>
      </div>
    </div>
    <div class="floating bottom">
      <TourWindow
        :tourStops="tourStops"
        :tourIndex="tourIndex"
        v-on:update:tourIndex="$emit('update:tourIndex', $event);"
      />
    </div>
    <ExplanationModal :currentModal.sync="currentModal" />
  </div>
</template>

<script>
import TourWindow from './TourWindow.vue'
import ExplanationModal from './ExplanationModal.vue'

import introContents from '../data/intro-contents.json';

export default {
  name: 'Controls',

  components: {
    TourWindow,
    ExplanationModal
  },

  props: {
    tourStops: {
      type: Array,
      default: () => []
    },

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
      introContents,

      sliderProperties: {
        minValue: null,
        maxValue: null,
        trackWidth: null,
        thumbDiameter: 20,
        labelWidth: 60
      },

      currentModal: false,

      availableTransportModes: {
        walking: 'Ich habe die Möglichkeit<br>zu Fuß zu gehen',
        cycling: 'Ich habe die Möglichkeit<br>Fahrrad zu fahren'
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
      }
    }
  },

  computed: {
    sliderProgressWidth() {
      const progressPercentage = (this.transportMinutes - this.sliderProperties.minValue) / (this.sliderProperties.maxValue - this.sliderProperties.minValue);
      const progressWidth = (this.sliderProperties.trackWidth - this.sliderProperties.thumbDiameter) * progressPercentage + this.sliderProperties.thumbDiameter / 2;

      return progressWidth;
    },
    sliderLabelOffset() {
      const labelOffset = this.sliderProgressWidth - this.sliderProperties.labelWidth / 2;

      return labelOffset;
    }
  },

  methods: {
    showIntro() {
      this.$emit('update:introScreen', true);
      this.$refs.sidebar.scrollTop = 0;
      window.localStorage.setItem('introScreen', 'true');
    },
    hideIntro() {
      this.$emit('update:introScreen', false);
      window.localStorage.setItem('introScreen', 'false');
    },
    startTour() {
      this.hideIntro();
      if (this.tourIndex === false) {
        setTimeout(() => {
          this.$emit('update:tourIndex', 0);
        }, 1000);
      }
    },
    exitTour() {
      this.$emit('update:tourIndex', false);
    },
    hideIntroAndExitTour() {
      this.hideIntro();
      this.exitTour();
    }
  },

  watch: {
    tourIndex: function(tourIndex) {
      if (tourIndex !== false) {
        this.$emit('update:contentView', this.tourStops[tourIndex].controlState.contentView);
        this.$emit('update:transportMode', this.tourStops[tourIndex].controlState.transportMode);
        this.$emit('update:transportMinutes', this.tourStops[tourIndex].controlState.transportMinutes);
        this.$emit('update:parkProperty', this.tourStops[tourIndex].controlState.parkProperty);
      }
    }
  },

  mounted () {
    const input = this.$refs.slider.getElementsByTagName('input')[0];
    this.sliderProperties.minValue = parseInt(input.getAttribute('min'));
    this.sliderProperties.maxValue = parseInt(input.getAttribute('max'));
    this.sliderProperties.trackWidth = input.getBoundingClientRect().width;
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

#app.intro .controls, .controls > * {
  pointer-events: auto;
}

/* Global focus styling */

.controls *:focus {
  outline: none;
}

.controls *:focus-visible {
  box-shadow: 0 0 0 2px #FAF6F0, 0 0 0 4px rgba(0,0,0,0.1);
}

/* Global properties of input, label and button elements */

.controls input, .controls label, .controls button {
  cursor: pointer;
  box-sizing: border-box;
}

/* No default styling of input elements and buttons */

.controls input, .controls button {
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
}

/* Text selection within labels and buttons invisible */

.controls label::selection, .controls label ::selection, .controls button::selection {
  background: transparent;
}

.controls label::-moz-selection, .controls label ::-moz-selection, .controls button::-moz-selection {
  background: transparent;
}

/* Hyphenation for all p elements */

.controls p {
  -webkit-hyphens: auto;
  -moz-hyphens: auto;
  hyphens: auto;
}


/* SIDEBAR GLOBAL */

.controls .sidebar {
  position: relative;
  width: 75vw;
  max-width: 250rem;
  height: 100%;
  background: #FAF6F0;
  box-shadow: 8rem 0rem 8rem rgba(0,0,0,0.03),
              4rem 0rem 4rem rgba(0,0,0,0.03),
              2rem 0rem 2rem rgba(0,0,0,0.03);
  transition: max-width 0.5s ease-in-out;
  z-index: 200;

  overflow-x: hidden;
  overflow-y: auto;
  scrollbar-width: none;
  -ms-overflow-style: none;
}

.controls .sidebar::-webkit-scrollbar {
  width: 0;
  height: 0;
}

#app.intro .controls .sidebar {
  max-width: 650rem;
  overflow-y: hidden;
}

.controls .sidebar .inner {
  transition: visibility 0.5s ease-in-out, opacity 0.5s ease-in-out;
}

.controls .sidebar .inner.intro,
#app.intro .controls .sidebar .inner.inputs {
  visibility: hidden;
  opacity: 0;
}

#app.intro .controls .sidebar .inner.intro {
  visibility: visible;
  opacity: 1;
}


/* SIDEBAR INPUTS VIEW: GLOBAL */

.controls .sidebar .inner.inputs {
  position: relative;
  box-sizing: border-box;
  max-width: 250rem;
  min-height: 100%;
  padding-bottom: 271rem;
}

.controls .sidebar .inner.inputs section {
  box-sizing: border-box;
  padding: 0 24rem;
}

.controls .sidebar .inner.inputs section.logo {
  padding-top: 24rem;
  text-align: center;
}

.controls .sidebar .inner.inputs section.reachability {
  margin-top: -4rem;
}

.controls .sidebar .inner.inputs section.properties {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  margin-bottom: 26rem;
  transition: visibility 0.1s ease-in-out, opacity 0.1s ease-in-out;
}

.controls .sidebar .inner.inputs section.properties.hidden {
  visibility: hidden;
  opacity: 0;
}

.controls .sidebar .inner.inputs h3 {
  display: inline-block;
  margin: 0 1rem 17rem;
  font-size: 12rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5rem;
  color: rgba(0,0,0,0.65);
}

.controls .sidebar .inner.inputs h3 + button {
  vertical-align: middle;
  width: 15rem;
  height: 15rem;
  margin-left: 5rem;
  border: none;
  border-radius: 50%;
  font-size: 0.1rem;
  color: transparent;
  background: url('../assets/icons/question.svg') center no-repeat rgba(0,0,0,0.1);
}

.controls .sidebar .inner.inputs .field {
  margin-top: 33rem;
}


/* SIDEBAR INPUTS VIEW: VERTICAL TOGGLE */

.controls .sidebar .vertical-toggle {
  position: relative;
}

.controls .sidebar .vertical-toggle:before {
  position: absolute;
  display: block;
  content: '';
  top: 0;
  bottom: 0;
  left: 0;
  width: 40rem;
  border-radius: 20rem;
  background: rgba(0,0,0,0.05);
  box-shadow: inset 0 0 0 2rem rgba(0,0,0,0.08);
}

.controls .sidebar .vertical-toggle label {
  position: relative;
  display: block;
}

.controls .sidebar .vertical-toggle label:before {
  position: absolute;
  display: block;
  content: '';
  top: 2rem;
  left: 10rem;
  width: 20rem;
  height: 2rem;
  background: rgba(0,0,0,0.11);
}

.controls .sidebar .vertical-toggle label:first-child:before,
.controls .sidebar .vertical-toggle label.checked:before,
.controls .sidebar .vertical-toggle label.checked + label:before {
  display: none;
}

.controls .sidebar .vertical-toggle label + label {
  margin-top: -6rem;
}

.controls .sidebar .vertical-toggle label input {
  opacity: 0.33;
  vertical-align: middle;
  margin: 0;
  width: 40rem;
  height: 52rem;
  border: 2rem solid transparent;
  border-radius: 20rem;
}

.controls .sidebar .vertical-toggle label input:checked {
  opacity: 1;
}

.controls .sidebar .vertical-toggle label input:focus-visible {
  box-shadow: 0 0 0 2px #FAF6F0, 0 0 0 4px #DFDBD5;
}

.controls .sidebar .vertical-toggle label span {
  display: inline-block;
  vertical-align: middle;
  max-width: 150rem;
  margin-left: 9rem;
  margin-right: -10rem;
  padding: 3rem 5rem;
  font-size: 12rem;
  font-weight: 600;
  line-height: 1.3;
  border-radius: 3rem;
  color: rgba(0,0,0,0.3);
}

.controls .sidebar .vertical-toggle label input:checked + span {
  color: rgba(0,0,0,0.75);
}

.controls .sidebar section.reachability .vertical-toggle:before {
  background: #F2FFD7;
  box-shadow: inset 0 0 0 2rem rgba(0,0,0,0.1);
}

.controls .sidebar section.reachability .vertical-toggle label + label {
  margin-top: -10rem;
}

.controls .sidebar section.reachability .vertical-toggle label input {
  height: 60rem;
}

.controls .sidebar section.reachability .vertical-toggle label input:checked {
  border-color: rgba(0,0,0,0.25);
}

.controls .sidebar section.reachability .vertical-toggle label.walking input {
  background: url('../assets/icons/walking.svg') center no-repeat;
}

.controls .sidebar section.reachability .vertical-toggle label.walking input:checked {
  background: url('../assets/icons/walking.svg') center no-repeat, linear-gradient(to bottom right, rgba(255, 255, 255, 0.6) 0%, rgba(255, 255, 255, 0) 100%), #C9FE53;
}

.controls .sidebar section.reachability .vertical-toggle label.cycling input {
  background: url('../assets/icons/cycling.svg') center no-repeat;
}

.controls .sidebar section.reachability .vertical-toggle label.cycling input:checked {
  background: url('../assets/icons/cycling.svg') center no-repeat, linear-gradient(to bottom right, rgba(255, 255, 255, 0.6) 0%, rgba(255, 255, 255, 0) 100%), #C9FE53;
}

.controls .sidebar section.reachability .vertical-toggle label input:checked + span {
  background: #C9FE53;
}

.controls .sidebar section.properties .vertical-toggle label.none input {
  background: url('../assets/icons/none.svg') center no-repeat;
}

.controls .sidebar section.properties .vertical-toggle label.none input:checked {
  border-color: #AFADAA;
  background: url('../assets/icons/none.svg') center no-repeat, linear-gradient(to bottom right, rgba(255, 255, 255, 0.2) 0%, rgba(255, 255, 255, 0) 100%), rgba(123, 123, 123, 0.15);
}

.controls .sidebar section.properties .vertical-toggle label.noise input {
  background: url('../assets/icons/noise.svg') center no-repeat;
}

.controls .sidebar section.properties .vertical-toggle label.noise input:checked {
  border-color: rgba(0,0,0,0.25);
  background: url('../assets/icons/noise-selected.svg') center no-repeat, linear-gradient(to bottom right, rgba(255, 255, 255, 0.2) 0%, rgba(255, 255, 255, 0) 100%), #E0580C;
}

.controls .sidebar section.properties .vertical-toggle label.noise input:checked + span {
  color: #E0580C;
}

.controls .sidebar section.properties .vertical-toggle label.vegetation input {
  background: url('../assets/icons/vegetation.svg') center no-repeat;
}

.controls .sidebar section.properties .vertical-toggle label.vegetation input:checked {
  border-color: rgba(0,0,0,0.25);
  background: url('../assets/icons/vegetation-selected.svg') center no-repeat, linear-gradient(to bottom right, rgba(255, 255, 255, 0.2) 0%, rgba(255, 255, 255, 0) 100%), #29955A;
}

.controls .sidebar section.properties .vertical-toggle label.vegetation input:checked + span {
  color: #29955A;
}


/* SIDEBAR INPUTS VIEW: SLIDER */

.controls .sidebar .slider {
  position: relative;
  margin-top: -4rem;
}

.controls .sidebar .slider span {
  position: absolute;
  display: block;
  top: 4rem;
  left: 0;
  height: 12rem;
  border-radius: 6rem 0 0 6rem;
  background: linear-gradient(to bottom right, rgba(255, 255, 255, 0.4) 0%, rgba(255, 255, 255, 0) 100%), #C9FE53;
  box-shadow: inset 0 0 0 2rem rgba(0,0,0,0.25);
  pointer-events: none;
}

/* Somewhat hacky way of hiding the progress span within Firefox (as Firefox does not support
   z-index on range’s shadow elements, but instead supplies additional progress element) */
@supports (-moz-appearance:none) {
  .controls .sidebar .slider span {
    display: none;
  }
}

.controls .sidebar .slider input {
  display: block;
  width: 100%;
  background: transparent;
}

.controls .sidebar .slider input:focus-visible {
  box-shadow: none;
}

/* Webkit range shadow elements */

.controls .sidebar .slider input::-webkit-slider-thumb,
.controls .sidebar .slider input::-webkit-slider-runnable-track {
  -webkit-appearance: none;
  appearance: none;
}

.controls .sidebar .slider input::-webkit-slider-thumb {
  cursor: grab;
  position: relative;
  width: 20rem;
  height: 20rem;
  margin-top: -4rem;
  border-radius: 100%;
  background: #565655;
  box-shadow: inset 0 0 0 2rem #565655, inset 0 0 0 4rem #C9FE53;
}

.controls .sidebar .slider input::-webkit-slider-thumb:active {
  cursor: grabbing;
}

.controls .sidebar .slider input::-webkit-slider-runnable-track {
  height: 12rem;
  margin: 4rem 0;
  border-radius: 6rem;
  background: #F2FFD7;
  box-shadow: inset 0 0 0 2rem rgba(0,0,0,0.1);
}

.controls .sidebar .slider input:focus-visible::-webkit-slider-runnable-track {
  box-shadow: inset 0 0 0 2rem rgba(0,0,0,0.1), 0 0 0 2px #FAF6F0, 0 0 0 4px rgba(0,0,0,0.1);
}

/* Firefox range shadow elements */

.controls .sidebar .slider input::-moz-range-thumb {
  cursor: grab;
  width: 20rem;
  height: 20rem;
  margin-top: -4rem;
  border: none;
  border-radius: 100%;
  background: #565655;
  box-shadow: inset 0 0 0 2rem #565655, inset 0 0 0 4rem #C9FE53;
}

.controls .sidebar .slider input::-moz-range-thumb:active {
  cursor: grabbing;
}

.controls .sidebar .slider input::-moz-range-track {
  height: 12rem;
  margin: 4rem 0;
  border-radius: 6rem;
  background: #F2FFD7;
  box-shadow: inset 0 0 0 2rem rgba(0,0,0,0.1);
}

.controls .sidebar .slider input:focus-visible::-moz-range-track {
  box-shadow: inset 0 0 0 2rem rgba(0,0,0,0.1), 0 0 0 2px #FAF6F0, 0 0 0 4px rgba(0,0,0,0.1);
}

.controls .sidebar .slider input::-moz-range-progress {
  height: 12rem;
  border-radius: 6rem 0 0 6rem;
  background: linear-gradient(to bottom right, rgba(255, 255, 255, 0.4) 0%, rgba(255, 255, 255, 0) 100%), #C9FE53;
  box-shadow: inset 0 0 0 2rem rgba(0,0,0,0.25);
}

/* End of range shadow elements */

.controls .sidebar .slider label {
  cursor: default;
  position: relative;
  display: inline-block;
  width: 60rem;
  margin-top: 10rem;
  text-align: center;
  font-size: 12rem;
  font-weight: 600;
  line-height: 14rem;
}

.controls .sidebar .slider label mark {
  padding: 3rem 5rem;
  border-radius: 3rem;
  color: inherit;
  background: #C9FE53;
}

.controls .sidebar .field.coverage {
  background: url('../assets/berlin-coverage.svg') 5rem -1rem no-repeat;
}

.controls .sidebar .field.coverage h3 {
  display: block;
  margin-top: 37rem;
  margin-bottom: 16rem;
}

.controls .sidebar .field.coverage mark {
  position: relative;
  display: inline-block;
  margin-left: -3rem;
  padding: 3rem 16rem;
  border-radius: 100rem;
  font-size: 32rem;
  font-weight: 600;
  font-feature-settings: 'tnum';
  letter-spacing: -2rem;
  color: rgba(0,0,0,0.75);
  background: #C9FE53;
}

.controls .sidebar .field.coverage mark span {
  margin-left: 4rem;
}

.controls .sidebar .field.coverage > span {
  display: block;
  margin: 16rem 0 0 1rem;
  font-size: 12rem;
}


/* SIDEBAR INTRO VIEW */

.controls .sidebar .inner.intro {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  width: 75vw;
  max-width: 650rem;
  overflow-x: hidden;
  overflow-y: auto;
}

.controls .sidebar .inner.intro section {
  max-width: 450rem;
  margin: 0 auto;
  padding: 0 25rem;
}

.controls .sidebar .inner.intro section.welcome {
  margin-top: 44rem;
  margin-bottom: 44rem;
  text-align: center;
}

.controls .sidebar .inner.intro section.welcome p {
  margin: 42rem 0 46rem;
  text-align: left;
  font-size: 14rem;
  line-height: 1.3;
}

.controls .sidebar .inner.intro section.welcome button {
  display: inline-block;
  padding: 0 16rem;
  font-size: 12rem;
  font-weight: 600;
  line-height: 36rem;
  border: 2rem solid rgba(0,0,0,0.1);
  border-radius: 20rem;
  color: rgba(0,0,0,0.65);
  background: rgba(0,0,0,0.06);
}

.controls .sidebar .inner.intro section.welcome button + button {
  margin-left: 12rem;
  border-color: rgba(0,0,0,0.2);
  color: rgba(0,0,0,0.75);
  background: linear-gradient(to bottom right, rgba(255, 255, 255, 0.3) 0%, rgba(255, 255, 255, 0) 100%), #C9FE53;
}


/* SIDEBAR CLOSE BUTTON */

.controls .sidebar button.close {
  visibility: hidden;
  opacity: 0;
  position: absolute;
  top: 12rem;
  right: 12rem;
  width: 28rem;
  height: 28rem;
  border: none;
  border-radius: 50%;
  background: url('../assets/icons/close.svg') center no-repeat rgba(0,0,0,0.06);
  border: 2rem solid rgba(0,0,0,0.1);
  transition: visibility 0.5s ease-in-out, opacity 0.5s ease-in-out;
}

#app.intro .controls .sidebar button.close {
  visibility: visible;
  opacity: 1;
}


/* FLOATING CONTROLS */

.controls .floating {
  position: absolute;
  left: 250rem;
  right: 0;
  text-align: center;
  pointer-events: none;
  transition: visibility 0.5s ease-in-out, opacity 0.5s ease-in-out;
  z-index: 150;
}

.controls .floating > * {
  pointer-events: auto;
}

#app.intro .controls .floating {
  visibility: hidden;
  opacity: 0;
}

.controls .floating.top {
  top: 0;
}

.controls .floating.bottom {
  bottom: 0;
}

.controls .floating.top .view-settings {
  display: inline-block;
  margin-top: 10rem;
  padding: 6rem;
  border-radius: 26rem;
  background: #FAF6F0;
  box-shadow: 0rem 8rem 8rem rgba(0, 0, 0, 0.03),
              0rem 4rem 4rem rgba(0, 0, 0, 0.03),
              0rem 2rem 2rem rgba(0, 0, 0, 0.03);
}

.controls .floating.top .view-settings button {
  display: inline-block;
  vertical-align: top;
  width: 40rem;
  height: 40rem;
  border: none;
  border-radius: 50%;
  background: url('../assets/icons/info.svg') center no-repeat rgba(0,0,0,0.06);
  border: 2rem solid rgba(0,0,0,0.1);
}

.controls .floating.top .view-settings .view-toggle {
  display: inline-block;
  vertical-align: top;
  margin-left: 10rem;
  border-radius: 20rem;
  background: rgba(0,0,0,0.06);
  box-shadow: inset 0 0 0 2rem rgba(0,0,0,0.1);
}

.controls .floating.top .view-settings .view-toggle input {
  width: 0;
  height: 0;
}

.controls .floating.top .view-settings .view-toggle label {
  display: inline-block;
  width: 92rem;
  font-size: 12rem;
  font-weight: 600;
  line-height: 36rem;
  border: 2rem solid transparent;
  border-radius: 20rem;
  color: rgba(0,0,0,0.65);
}

.controls .floating.top .view-settings .view-toggle label + input + label {
  margin-left: -4rem;
}

.controls .floating.top .view-settings .view-toggle input:checked + label {
  border-color: rgba(0,0,0,0.22);
  color: #fff;
  background: linear-gradient(to bottom right, rgba(255, 255, 255, 0.1) 0%, rgba(255, 255, 255, 0) 100%), #7D7D7D;
}

.controls .floating.top .view-settings .view-toggle input:focus-visible {
  box-shadow: none;
}

.controls .floating.top .view-settings .view-toggle input:focus-visible + label {
  box-shadow: 0 0 0 2px #FAF6F0, 0 0 0 4px #DFDBD5;
}
</style>