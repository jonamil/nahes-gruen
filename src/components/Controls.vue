<template>
  <div class="controls">
    <div class="sidebar">
      <div class="inner">
        <section class="logo">
          <img src="../assets/logo.svg" height="86" />
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
    </div>
    <div class="floating top">
      <div class="view-settings">
        <button title="Einführung" @click="$emit('update:introScreen', true)" />
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

.controls > * {
  pointer-events: auto;
}

.controls label::selection, .controls label ::selection, .controls button::selection {
  background: transparent;
}

.controls label::-moz-selection, .controls label ::-moz-selection, .controls button::-moz-selection {
  background: transparent;
}

.controls .sidebar {
  width: 250rem;
  height: 100%;
  background: #FAF6F0;
  box-shadow: 8rem 0rem 8rem rgba(0,0,0,0.03),
              4rem 0rem 4rem rgba(0,0,0,0.03),
              2rem 0rem 2rem rgba(0,0,0,0.03);

  overflow-y: auto;
  scrollbar-width: none;
  -ms-overflow-style: none;
}

.controls .sidebar::-webkit-scrollbar {
  width: 0;
  height: 0;
}

.controls .sidebar .inner {
  position: relative;
  box-sizing: border-box;
  min-height: 100%;
  padding-bottom: 239rem;
}

.controls section {
  padding: 0 24rem;
}

.controls section.logo {
  padding-top: 24rem;
  text-align: center;
}

.controls section.reachability {
  margin-top: -4rem;
}

.controls section.properties {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  margin-bottom: 26rem;
  transition: visibility 0.1s ease-in-out, opacity 0.1s ease-in-out;
}

.controls section.properties.hidden {
  visibility: hidden;
  opacity: 0;
}

.controls .sidebar h3 {
  display: inline-block;
  margin: 0 1rem 17rem;
  font-size: 12rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5rem;
  color: rgba(0,0,0,0.65);
}

.controls .sidebar h3 + button {
  cursor: pointer;
  appearance: none;
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

.controls .sidebar h3 + button:focus {
  outline: none;
}

.controls .sidebar h3 + button:focus-visible {
  box-shadow: 0 0 0 2px #FAF6F0, 0 0 0 4px rgba(0,0,0,0.1);
}

.controls .sidebar .field {
  margin-top: 33rem;
}

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
  cursor: pointer;
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
  -webkit-appearance: none;
  appearance: none;
  cursor: pointer;
  opacity: 0.33;
  vertical-align: middle;
  box-sizing: border-box;
  margin: 0;
  width: 40rem;
  height: 52rem;
  border-radius: 20rem;
}

.controls .sidebar .vertical-toggle label input:checked {
  opacity: 1;
}

.controls .sidebar .vertical-toggle label input:focus {
  outline: none;
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

.controls .sidebar section.reachability .vertical-toggle:focus-within:before {
  /*box-shadow: inset 0 0 0 2rem rgba(0,0,0,0.1), 0 0 0 2px #FAF6F0, 0 0 0 4px rgba(0,0,0,0.1);*/
}

.controls .sidebar section.reachability .vertical-toggle label + label {
  margin-top: -10rem;
}

.controls .sidebar section.reachability .vertical-toggle label input {
  height: 60rem;
}

.controls .sidebar section.reachability .vertical-toggle label input:checked {
  border: 2rem solid rgba(0,0,0,0.25);
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
  border: 2rem solid #AFADAA;
  background: url('../assets/icons/none.svg') center no-repeat, linear-gradient(to bottom right, rgba(255, 255, 255, 0.2) 0%, rgba(255, 255, 255, 0) 100%), rgba(123, 123, 123, 0.15);
}

.controls .sidebar section.properties .vertical-toggle label.noise input {
  background: url('../assets/icons/noise.svg') center no-repeat;
}

.controls .sidebar section.properties .vertical-toggle label.noise input:checked {
  border: 2rem solid rgba(0,0,0,0.25);
  background: url('../assets/icons/noise-selected.svg') center no-repeat, linear-gradient(to bottom right, rgba(255, 255, 255, 0.2) 0%, rgba(255, 255, 255, 0) 100%), #E0580C;
}

.controls .sidebar section.properties .vertical-toggle label.noise input:checked + span {
  color: #E0580C;
}

.controls .sidebar section.properties .vertical-toggle label.vegetation input {
  background: url('../assets/icons/vegetation.svg') center no-repeat;
}

.controls .sidebar section.properties .vertical-toggle label.vegetation input:checked {
  border: 2rem solid rgba(0,0,0,0.25);
  background: url('../assets/icons/vegetation-selected.svg') center no-repeat, linear-gradient(to bottom right, rgba(255, 255, 255, 0.2) 0%, rgba(255, 255, 255, 0) 100%), #29955A;
}

.controls .sidebar section.properties .vertical-toggle label.vegetation input:checked + span {
  color: #29955A;
}

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
  appearance: none;
  cursor: pointer;
  display: block;
  width: 100%;
  background: transparent;
}

.controls .sidebar .slider input:focus {
  outline: none;
}

/* WEBKIT range shadow elements */

.controls .sidebar .slider input,
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

/* FIREFOX range shadow elements */

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

/* END of range shadow elements */

.controls .sidebar .slider label {
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

.controls .floating {
  position: absolute;
  left: 250rem;
  right: 0;
  text-align: center;
  pointer-events: none;
}

.controls .floating > * {
  pointer-events: auto;
}

.controls .floating.top {
  top: 0;
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
  -webkit-appearance: none;
  appearance: none;
  cursor: pointer;
  display: inline-block;
  vertical-align: top;
  box-sizing: border-box;
  width: 40rem;
  height: 40rem;
  border: none;
  border-radius: 50%;
  background: url('../assets/icons/info.svg') center no-repeat rgba(0,0,0,0.06);
  border: 2rem solid rgba(0,0,0,0.1);
}

.controls .floating.top .view-settings button:focus {
  outline: none;
}

.controls .floating.top .view-settings button:focus-visible {
  box-shadow: 0 0 0 2px #FAF6F0, 0 0 0 4px rgba(0,0,0,0.1);
}

.controls .floating.top .view-settings .view-toggle {
  display: inline-block;
  vertical-align: top;
  margin-left: 10rem;
  border-radius: 25rem;
  background: rgba(0,0,0,0.06);
  box-shadow: inset 0 0 0 2rem rgba(0,0,0,0.1);
}

.controls .floating.top .view-settings .view-toggle:focus-within {
  /*box-shadow: inset 0 0 0 2rem rgba(0,0,0,0.1), 0 0 0 2px #FAF6F0, 0 0 0 4px rgba(0,0,0,0.1);*/
}

.controls .floating.top .view-settings .view-toggle input {
  -webkit-appearance: none;
  appearance: none;
  width: 0;
  height: 0;
}

.controls .floating.top .view-settings .view-toggle label {
  cursor: pointer;
  display: inline-block;
  box-sizing: border-box;
  width: 92rem;
  font-size: 12rem;
  font-weight: 600;
  line-height: 36rem;
  border: 2rem solid transparent;
  border-radius: 25rem;
  color: rgba(0,0,0,0.65);
}

.controls .floating.top .view-settings .view-toggle label + input + label {
  margin-left: -4rem;
}

.controls .floating.top .view-settings .view-toggle input:checked + label {
  border: 2rem solid rgba(0,0,0,0.22);
  color: #fff;
  background: linear-gradient(to bottom right, rgba(255, 255, 255, 0.1) 0%, rgba(255, 255, 255, 0) 100%), #7D7D7D;
}

.controls .floating.top .view-settings .view-toggle input:focus-visible + label {
  box-shadow: 0 0 0 2px #FAF6F0, 0 0 0 4px #DFDBD5;
}

.controls .floating.bottom {
  bottom: 0;
}
</style>