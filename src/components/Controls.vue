<template>
  <div class="controls" @click.self="hideIntro">
    <div ref="sidebar" class="sidebar">
      <div class="inner inputs">
        <section class="logo">
          <img class="large" src="../assets/logo.svg" height="86" alt="Nahes Grün, Fernes Grün" @click="showIntro" />
          <img class="small" src="../assets/logo-small.svg" height="64" alt="Nahes Grün, Fernes Grün" @click="showIntro" />
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
              <span :style="sliderProperties.minValue ? 'width: ' + sliderProgressWidth / 10 + 'rem' : ''" />
              <input
                id="transport-minutes"
                type="range"
                min="1"
                max="15"
                step="1"
                :value="transportMinutes"
                @input="$emit('update:transportMinutes', parseInt($event.target.value))"
              >
              <label for="transport-minutes" :style="sliderProperties.minValue ? 'left: ' + sliderLabelOffset / 10 + 'rem' : ''">
                <mark>{{ transportMinutes }} Min.</mark>
              </label>
            </div>
          </div>
          <div class="field coverage">
            <h3>Abdeckung</h3>
            <button title="Erläuterung" @click="currentModal = 'coverage'" />
            <br><mark>{{ coveragePercentage }}<span>%</span></mark>
            <p>aller Berliner*innen können unter diesen Bedingungen von zuhause einen Grünraum erreichen.</p>
          </div>
        </section>
        <section :class="'properties' + (contentView !== 'map' ? ' hidden' : '')">
          <div class="field">
            <h3>Flächenmerkmale</h3>
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
          <img src="../assets/logo.svg" height="126" alt="Nahes Grün, Fernes Grün" />
          <p v-html="introContents.welcome" />
          <button @click="exploreMap">Karte erkunden</button>
          <button @click="startTour">
            {{ tourIndex !== false ? 'Rundgang fortsetzen' : 'Rundgang starten' }}
          </button>
        </section>
        <section ref="introArticle" class="article">
          <div
            v-for="(section, index) in introContents.article"
            :key="index"
          >
            <h3 v-if="section.heading">{{ section.heading }}</h3>
            <p
              v-for="(paragraph, index) in section.paragraphs"
              :key="index"
              v-html="paragraph"
            />
          </div>
        </section>
        <section class="footer">
          <div class="fade" />
          <button class="expand" @click="toggleArticle">
            {{ introExpanded ? 'Text einklappen' : 'Weiterlesen' }}
          </button>
          <h3>Team</h3>
          <button class="info" title="Erläuterung" @click="currentModal = 'team'" />
          <p>
            <span
              v-for="(name, index) in introContents.team"
              :key="index"
              v-html="name + ' '"
            />
          </p>
          <h3>Über das Projekt</h3>
          <p v-html="introContents.about" />
          <a target="_blank" href="https://fh-potsdam.de/">
            <img src="../assets/fhp.svg" alt="Fachhochschule Potsdam" />
          </a>
        </section>
      </div>
      <button class="close" title="Schließen" @click="hideIntro" />
    </div>
    <div class="floating inputs top">
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
    <div class="floating inputs bottom">
      <TourWindow
        :tourStops="tourStops"
        :tourIndex="tourIndex"
        v-on:update:tourIndex="$emit('update:tourIndex', $event)"
        v-on:update:currentModal="currentModal = $event"
      />
    </div>
    <div class="floating intro bottom">
      <ContrastWindow
        :higherContrast="higherContrast"
        v-on:update:higherContrast="$emit('update:higherContrast', $event)"
      />
    </div>
    <ExplanationModal :currentModal.sync="currentModal" />
  </div>
</template>

<script>
import TourWindow from './TourWindow.vue';
import ContrastWindow from './ContrastWindow.vue';
import ExplanationModal from './ExplanationModal.vue';

import introContents from '../data/intro-contents.json';

export default {
  name: 'Controls',

  components: {
    TourWindow,
    ContrastWindow,
    ExplanationModal
  },

  props: {
    tourStops: {
      type: Array,
      default: () => []
    },
    coverage: {
      type: Object,
      default: () => {}
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
    higherContrast: {
      type: Boolean,
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
      introExpanded: false,

      availableTransportModes: {
        walking: 'Ich habe die Möglichkeit<br>zu Fuß zu gehen',
        cycling: 'Ich habe die Möglichkeit<br>Fahrrad zu fahren'
      },
      availableParkProperties: {
        none: {
          title: 'Keine Auswahl'
        },
        noise: {
          title: 'Lärmbelastung'
        },
        vegetation: {
          title: 'Vegetationsmenge'
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
    },
    coveragePercentage() {
      const percentage = this.coverage.city.percentages[this.transportMode][this.transportMinutes.toString()];

      let digits = 1;
      if (percentage >= 99.95) digits = 0;

      return new Intl.NumberFormat('de-DE', { minimumFractionDigits: digits, maximumFractionDigits: digits }).format(percentage);
    }
  },

  methods: {
    showIntro() {
      this.$emit('update:introScreen', true);
      this.$refs.sidebar.scrollTop = 0;
    },
    hideIntro() {
      this.$emit('update:introScreen', false);
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
    exploreMap() {
      this.exitTour();
      this.hideIntro();
      this.$emit('update:contentView', 'map');
    },
    toggleArticle() {
      if (this.introExpanded) this.$refs.intro.scrollTop = 0;
      this.introExpanded = !this.introExpanded;
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

    this.$refs.introArticle.addEventListener('click', event => {
      if (event.target.localName === 'sup' || (event.target.localName === 'a' && event.target.classList.contains('sources'))) {
        this.currentModal = 'sources';
        event.preventDefault();
      }
    });
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

/* Hyphenation for all p elements and reference number style */

.controls p {
  -webkit-hyphens: auto;
  -moz-hyphens: auto;
  hyphens: auto;
}

.controls p sup {
  cursor: pointer;
  top: -0.4rem;
  margin: 0 0 -0.1rem 0.1rem;
  padding: 0.1rem 0.2rem;
  font-size: 0.7em;
  /*font-size: inherit;*/
  border-radius: 0.2rem;
  color: rgba(0,0,0,0.5);
  background: rgba(0,0,0,0.1);
}


/* SIDEBAR GLOBAL */

.controls .sidebar {
  position: relative;
  width: 75vw;
  max-width: 25.0rem;
  height: 100%;
  background: #FAF6F0;
  box-shadow: 0.8rem 0 0.8rem rgba(0,0,0,0.03),
              0.4rem 0 0.4rem rgba(0,0,0,0.03),
              0.2rem 0 0.2rem rgba(0,0,0,0.03);
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
  max-width: 65.0rem;
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
  max-width: 25.0rem;
  min-height: 100%;
  padding-bottom: 27.1rem;
}

.controls .sidebar .inner.inputs section {
  box-sizing: border-box;
  padding: 0 2.4rem;
}

.controls .sidebar .inner.inputs section.logo {
  margin-bottom: 0.8rem;
  padding-top: 2.4rem;
  text-align: center;
}

.controls .sidebar .inner.inputs section.reachability {
  padding-top: 2.1rem;
}

.controls .sidebar .inner.inputs section.properties {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  margin-bottom: 2.6rem;
  transition: visibility 0.1s ease-in-out, opacity 0.1s ease-in-out;
}

.controls .sidebar .inner.inputs section.properties.hidden {
  visibility: hidden;
  opacity: 0;
}

.controls .sidebar .inner.inputs h3 {
  display: inline-block;
  margin: 0 0.1rem 1.7rem;
  font-size: 1.2rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05rem;
  color: rgba(0,0,0,0.65);
}

.controls .sidebar .inner.inputs h3 + button {
  vertical-align: middle;
  width: 1.5rem;
  height: 1.5rem;
  margin-left: 0.5rem;
  border: none;
  border-radius: 50%;
  color: transparent;
  background: url('../assets/icons/question.svg') center no-repeat rgba(0,0,0,0.1);
}

.controls .sidebar .inner.inputs .field {
  margin-top: 3.3rem;
}

.controls .sidebar .inner.inputs section.reachability .field:first-child {
  margin-top: 0;
}


/* LOGO */

.controls .sidebar .inner.inputs section.logo img {
  cursor: pointer;
}

.controls .sidebar .inner.inputs section.logo img.small {
  display: none;
  margin: -0.2rem 0 -1.1rem -0.5rem;
}

@media (max-height: 844px) {
  .controls .sidebar .inner.inputs section.logo img.large {
    display: none;
  }

  .controls .sidebar .inner.inputs section.logo img.small {
    display: initial;
  }
}

@media (max-height: 809px) {
  .controls .sidebar .inner.inputs section.logo {
    display: none;
  }
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
  width: 4.0rem;
  border-radius: 2.0rem;
  background: rgba(0,0,0,0.05);
  box-shadow: inset 0 0 0 0.2rem rgba(0,0,0,0.08);
}

.controls .sidebar .vertical-toggle label {
  position: relative;
  display: block;
}

.controls .sidebar .vertical-toggle label:before {
  position: absolute;
  display: block;
  content: '';
  top: 0.2rem;
  left: 1.0rem;
  width: 2.0rem;
  height: 0.2rem;
  background: rgba(0,0,0,0.11);
}

.controls .sidebar .vertical-toggle label:first-child:before,
.controls .sidebar .vertical-toggle label.checked:before,
.controls .sidebar .vertical-toggle label.checked + label:before {
  display: none;
}

.controls .sidebar .vertical-toggle label + label {
  margin-top: -0.6rem;
}

.controls .sidebar .vertical-toggle label input {
  opacity: 0.33;
  vertical-align: middle;
  margin: 0;
  width: 4.0rem;
  height: 5.2rem;
  border: 0.2rem solid transparent;
  border-radius: 2.0rem;
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
  max-width: 15.0rem;
  margin-left: 0.9rem;
  margin-right: -1.0rem;
  padding: 0.3rem 0.5rem;
  font-size: 1.2rem;
  font-weight: 600;
  line-height: 1.3;
  border-radius: 0.3rem;
  color: rgba(0,0,0,0.3);
}

.controls .sidebar .vertical-toggle label input:checked + span {
  color: rgba(0,0,0,0.75);
}

.controls .sidebar section.reachability .vertical-toggle:before {
  background: #F2FFD7;
  box-shadow: inset 0 0 0 0.2rem rgba(0,0,0,0.1);
}

.controls .sidebar section.reachability .vertical-toggle label + label {
  margin-top: -1.0rem;
}

.controls .sidebar section.reachability .vertical-toggle label input {
  height: 6.0rem;
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
  background: url('../assets/icons/noise.svg') center no-repeat, linear-gradient(to bottom right, rgba(255, 255, 255, 0.2) 0%, rgba(255, 255, 255, 0) 100%), #F4A24F;
}

.controls .sidebar section.properties .vertical-toggle label.noise input:checked + span {
  /*color: #E0580C;*/
}

.controls .sidebar section.properties .vertical-toggle label.vegetation input {
  background: url('../assets/icons/vegetation.svg') center no-repeat;
}

.controls .sidebar section.properties .vertical-toggle label.vegetation input:checked {
  border-color: rgba(0,0,0,0.25);
  background: url('../assets/icons/vegetation.svg') center no-repeat, linear-gradient(to bottom right, rgba(255, 255, 255, 0.2) 0%, rgba(255, 255, 255, 0) 100%), #5FBF7A;
}

.controls .sidebar section.properties .vertical-toggle label.vegetation input:checked + span {
  /*color: #29955A;*/
}


/* SIDEBAR INPUTS VIEW: SLIDER */

.controls .sidebar .slider {
  position: relative;
  margin-top: -0.4rem;
}

.controls .sidebar .slider span {
  position: absolute;
  display: block;
  top: 0.4rem;
  left: 0;
  height: 1.2rem;
  border-radius: 0.6rem 0 0 0.6rem;
  background: linear-gradient(to bottom right, rgba(255, 255, 255, 0.4) 0%, rgba(255, 255, 255, 0) 100%), #C9FE53;
  box-shadow: inset 0 0 0 0.2rem rgba(0,0,0,0.25);
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
  width: 2.0rem;
  height: 2.0rem;
  margin-top: -0.4rem;
  border-radius: 100%;
  background: #565655;
  box-shadow: inset 0 0 0 0.2rem #565655, inset 0 0 0 0.4rem #C9FE53;
}

.controls .sidebar .slider input::-webkit-slider-thumb:active {
  cursor: grabbing;
}

.controls .sidebar .slider input::-webkit-slider-runnable-track {
  height: 1.2rem;
  margin: 0.4rem 0;
  border-radius: 0.6rem;
  background: #F2FFD7;
  box-shadow: inset 0 0 0 0.2rem rgba(0,0,0,0.1);
}

.controls .sidebar .slider input:focus-visible::-webkit-slider-runnable-track {
  box-shadow: inset 0 0 0 0.2rem rgba(0,0,0,0.1), 0 0 0 2px #FAF6F0, 0 0 0 4px rgba(0,0,0,0.1);
}

/* Firefox range shadow elements */

.controls .sidebar .slider input::-moz-range-thumb {
  cursor: grab;
  width: 2.0rem;
  height: 2.0rem;
  margin-top: -0.4rem;
  border: none;
  border-radius: 100%;
  background: #565655;
  box-shadow: inset 0 0 0 0.2rem #565655, inset 0 0 0 0.4rem #C9FE53;
}

.controls .sidebar .slider input::-moz-range-thumb:active {
  cursor: grabbing;
}

.controls .sidebar .slider input::-moz-range-track {
  height: 1.2rem;
  margin: 0.4rem 0;
  border-radius: 0.6rem;
  background: #F2FFD7;
  box-shadow: inset 0 0 0 0.2rem rgba(0,0,0,0.1);
}

.controls .sidebar .slider input:focus-visible::-moz-range-track {
  box-shadow: inset 0 0 0 0.2rem rgba(0,0,0,0.1), 0 0 0 2px #FAF6F0, 0 0 0 4px rgba(0,0,0,0.1);
}

.controls .sidebar .slider input::-moz-range-progress {
  height: 1.2rem;
  border-radius: 0.6rem 0 0 0.6rem;
  background: linear-gradient(to bottom right, rgba(255, 255, 255, 0.4) 0%, rgba(255, 255, 255, 0) 100%), #C9FE53;
  box-shadow: inset 0 0 0 0.2rem rgba(0,0,0,0.25);
}

/* End of range shadow elements */

.controls .sidebar .slider label {
  cursor: default;
  position: relative;
  display: inline-block;
  width: 6.0rem;
  margin-top: 1.0rem;
  text-align: center;
  font-size: 1.2rem;
  font-weight: 600;
  line-height: 1.4rem;
}

.controls .sidebar .slider label mark {
  padding: 0.3rem 0.6rem 0.3rem 0.7rem;
  color: inherit;
  background: url('../assets/edges/left-ui-4.svg') left -0.1rem center no-repeat, url('../assets/edges/right-ui-3.svg') right -0.1rem center no-repeat, #C9FE53;
  background-size: auto 100%;
}

.controls .sidebar .field.coverage {
  background: url('../assets/berlin-coverage.svg') 0.5rem 0.2rem no-repeat;
}

.controls .sidebar .field.coverage h3 + button {
  background-color: #E1DDD8;
  box-shadow: 0 0 0 2px #FAF6F0;
}

.controls .sidebar .field.coverage h3 + button:focus-visible {
  box-shadow: 0 0 0 2px #FAF6F0, 0 0 0 4px rgba(0,0,0,0.1);
}

.controls .sidebar .field.coverage mark {
  position: relative;
  display: inline-block;
  margin: -0.1rem 0 0 -0.3rem;
  padding: 0.2rem 1.4rem 0.2rem 1.3rem;
  font-size: 3.2rem;
  font-weight: 600;
  font-feature-settings: 'tnum';
  letter-spacing: -0.2rem;
  color: rgba(0,0,0,0.75);
  background: url('../assets/edges/left-ui-1.svg') left -0.1rem center no-repeat, url('../assets/edges/right-bg-3.svg') right -0.1rem center no-repeat, #C9FE53;
  background-size: auto 100%;
}

.controls .sidebar .field.coverage mark span {
  margin-left: 0.4rem;
}

.controls .sidebar .field.coverage p {
  display: block;
  margin: 1.6rem 0 0 0.1rem;
  font-size: 1.2rem;
}


/* SIDEBAR INTRO VIEW */

.controls .sidebar .inner.intro {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  width: 75vw;
  max-width: 65.0rem;
  overflow: hidden;
}

.controls .sidebar .inner.intro.expanded {
  overflow-y: auto;
}

.controls .sidebar .inner.intro h3 {
  margin: 2.2rem 0.1rem -0.1rem 0;
  font-size: 1.3rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05rem;
  color: rgba(0,0,0,0.65);
}

.controls .sidebar .inner.intro p {
  margin: 0.9rem 0 0;
  font-size: 1.2rem;
  line-height: 1.5;
}

.controls .sidebar .inner.intro p a {
  padding-bottom: 0.1rem;
  text-decoration: none;
  box-shadow: inset 0 -0.15rem rgba(0,0,0,0.2);
  color: inherit;
  transition: box-shadow 0.1s ease-in-out;
}

.controls .sidebar .inner.intro p a:hover {
  box-shadow: inset 0 -0.15rem rgba(0,0,0,0.65);
}

.controls .sidebar .inner.intro p figure {
  position: relative;
  margin: 0;
}

.controls .sidebar .inner.intro p figure img {
  display: block;
  width: 100%;
  margin: 2.5rem 0 2.7rem;
}

.controls .sidebar .inner.intro p figure figcaption {
  position: absolute;
  left: -1.3rem;
  bottom: 0;
  font-size: 1.0rem;
  line-height: 1.2;
  color: rgba(0,0,0,0.4);
  transform: rotate(-90deg);
  transform-origin: bottom left;
}

.controls .sidebar .inner.intro section {
  max-width: 45.0rem;
  padding: 0 2.5rem;
  margin: 0 auto;
}

.controls .sidebar .inner.intro section.welcome {
  margin-top: 3.8rem;
  text-align: center;
}

.controls .sidebar .inner.intro section.welcome p {
  margin: 3.4rem 0 3.9rem;
  text-align: left;
  font-size: 1.5rem;
  font-weight: 520;
  line-height: 1.35;
}

.controls .sidebar .inner.intro section.welcome button {
  display: inline-block;
  padding: 0 1.6rem;
  font-size: 1.2rem;
  font-weight: 600;
  line-height: 3.6rem;
  border: 0.2rem solid rgba(0,0,0,0.1);
  border-radius: 2.0rem;
  color: rgba(0,0,0,0.65);
  background: rgba(0,0,0,0.06);
}

.controls .sidebar .inner.intro section.welcome button + button {
  margin-left: 1.2rem;
  border-color: rgba(0,0,0,0.2);
  color: rgba(0,0,0,0.75);
  background: linear-gradient(to bottom right, rgba(255, 255, 255, 0.3) 0%, rgba(255, 255, 255, 0) 100%), #C9FE53;
}

.controls .sidebar .inner.intro section.article {
  margin-top: 9.7rem;
}

.controls .sidebar .inner.intro section.footer {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 0 2.5rem 4.0rem;
  background: #FAF6F0;
}

.controls .sidebar .inner.intro.expanded section.footer {
  position: relative;
  margin-top: 4.65rem;
}

.controls .sidebar .inner.intro section.footer .fade {
  position: absolute;
  bottom: 100%;
  left: 0;
  right: 0;
  height: 9.0rem;
  background-image: linear-gradient(to bottom, rgba(250,246,240,0) 0%, rgba(250,246,240,1) 100%);
  pointer-events: none;
}

.controls .sidebar .inner.intro.expanded section.footer .fade {
  display: none;
}

.controls .sidebar .inner.intro section.footer button.expand {
  display: block;
  margin: 0.5rem auto 0.5rem;
  padding: 0 1.2rem;
  font-size: 1.2rem;
  font-weight: 600;
  line-height: 2.6rem;
  border: 0.2rem solid rgba(0,0,0,0.1);
  border-radius: 1.5rem;
  color: rgba(0,0,0,0.65);
  background: transparent;
}

.controls .sidebar .inner.intro section.footer h3 {
  display: inline-block;
}

.controls .sidebar .inner.intro section.footer p {
  margin-top: 0.7rem;
  line-height: 1.45;
}

.controls .sidebar .inner.intro section.footer p span {
  margin-right: 0.6rem;
}

.controls .sidebar .inner.intro section.footer p span:after {
  content: '·';
  font-weight: 700;
  margin-left: 0.3rem;
}

.controls .sidebar .inner.intro section.footer p span:last-child:after {
  display: none;
}

.controls .sidebar .inner.intro section.footer > a {
  position: absolute;
  left: -3.4rem;
  bottom: 4.0rem;
  opacity: 0.4;
  transition: opacity 0.1s ease-in-out;
}

.controls .sidebar .inner.intro section.footer > a:hover {
  opacity: 0.85;
}

@media (max-height: 792px) {
  .controls .sidebar .inner.intro section.welcome {
    margin-top: 3.3rem;
  }

  .controls .sidebar .inner.intro section.welcome img {
    height: 10.0rem;
  }

  .controls .sidebar .inner.intro section.welcome p {
    margin: 3.0rem 0 3.5rem;
  }

  .controls .sidebar .inner.intro section.article {
    margin-top: 7.7rem;
  }
}

@media (max-height: 720px) {
  .controls .sidebar .inner.intro:not(.expanded) section.footer {
    top: 49.8rem;
    bottom: initial;
  }
}


/* SIDEBAR CLOSE BUTTON */

.controls .sidebar button.close {
  visibility: hidden;
  opacity: 0;
  position: absolute;
  top: 1.2rem;
  right: 1.2rem;
  width: 2.8rem;
  height: 2.8rem;
  border: none;
  border-radius: 50%;
  background: url('../assets/icons/close.svg') center no-repeat rgba(0,0,0,0.06);
  border: 0.2rem solid rgba(0,0,0,0.1);
  transition: visibility 0.5s ease-in-out, opacity 0.5s ease-in-out;
}

#app.intro .controls .sidebar button.close {
  visibility: visible;
  opacity: 1;
}


/* FLOATING CONTROLS */

.controls .floating {
  position: absolute;
  left: 25.0rem;
  right: 0;
  text-align: center;
  pointer-events: none;
  transition: visibility 0.5s ease-in-out, opacity 0.5s ease-in-out;
  z-index: 150;
}

.controls .floating > * {
  pointer-events: auto;
}

.controls .floating.intro {
  visibility: hidden;
  opacity: 0;
  left: 65.0rem;
}

#app.intro .controls .floating.inputs {
  visibility: hidden;
  opacity: 0;
}

#app.intro .controls .floating.inputs * {
  pointer-events: none;
}

#app.intro .controls .floating.intro {
  visibility: visible;
  opacity: 1;
}

.controls .floating.top {
  top: 0;
}

.controls .floating.bottom {
  bottom: 0;
}

.controls .floating.top .view-settings {
  display: inline-block;
  margin-top: 1.0rem;
  padding: 0.6rem;
  border-radius: 2.6rem;
  background: #FAF6F0;
  box-shadow: 0 0.8rem 0.8rem rgba(0, 0, 0, 0.03),
              0 0.4rem 0.4rem rgba(0, 0, 0, 0.03),
              0 0.2rem 0.2rem rgba(0, 0, 0, 0.03);
}

.controls .floating.top .view-settings button {
  display: inline-block;
  vertical-align: top;
  width: 4.0rem;
  height: 4.0rem;
  border: none;
  border-radius: 50%;
  background: url('../assets/icons/info.svg') center no-repeat rgba(0,0,0,0.06);
  border: 0.2rem solid rgba(0,0,0,0.1);
}

.controls .floating.top .view-settings .view-toggle {
  display: inline-block;
  vertical-align: top;
  margin-left: 1.0rem;
  border-radius: 2.0rem;
  background: rgba(0,0,0,0.06);
  box-shadow: inset 0 0 0 0.2rem rgba(0,0,0,0.1);
}

.controls .floating.top .view-settings .view-toggle input {
  width: 0;
  height: 0;
}

.controls .floating.top .view-settings .view-toggle label {
  display: inline-block;
  width: 9.2rem;
  font-size: 1.2rem;
  font-weight: 600;
  line-height: 3.6rem;
  border: 0.2rem solid transparent;
  border-radius: 2.0rem;
  color: rgba(0,0,0,0.65);
}

.controls .floating.top .view-settings .view-toggle label + input + label {
  margin-left: -0.4rem;
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