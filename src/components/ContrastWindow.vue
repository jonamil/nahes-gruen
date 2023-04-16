<template>
  <div class="contrast" :class="{ open: open }" @click.self="open = false">
    <div v-show="!open" class="collapsed">
      <button @click="open = true">Kontraste</button>
    </div>
    <div v-show="open" class="expanded">
      <button title="Schließen" @click="open = false" />
      <h3>Kontraste</h3>
      <p>Hier können die Kontraste der Karte erhöht werden. Dies ist empfehlenswert für Menschen mit Beeinträchtigungen der Farbwahrnehmung.</p>
      <div class="vertical-toggle">
        <label class="standard" :class="{ checked: !higherContrast }">
          <input
            type="radio"
            name="higher-contrast"
            :checked="!higherContrast"
            @change="$emit('update:higherContrast', false)"
          />
          <span>Standardansicht</span>
        </label>
        <label class="higher" :class="{ checked: higherContrast }">
          <input
            type="radio"
            name="higher-contrast"
            :checked="higherContrast"
            @change="$emit('update:higherContrast', true)"
          />
          <span>Erhöhte Kontraste</span>
        </label>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ContrastWindow',

  props: {
    higherContrast: {
      type: Boolean,
      default: false
    }
  },

  data () {
    return {
      open: false
    }
  }
}
</script>

<style>
.contrast {
  pointer-events: none !important;
}

.contrast.open, .contrast > * {
  pointer-events: auto !important;
}

.contrast button:focus-visible {
  box-shadow: 0 0 0 2px #262524, 0 0 0 4px #60ABF0;
}

.contrast .collapsed {
  display: inline-block;
  margin-bottom: 2.0rem;
  padding: 0.6rem;
  border-radius: 2.6rem;
  background: #262524;
  box-shadow: 0 0.8rem 0.8rem rgba(0,0,0,0.05),
              0 0.4rem 0.4rem rgba(0,0,0,0.05),
              0 0.2rem 0.2rem rgba(0,0,0,0.05);
}

.contrast .collapsed button {
  display: inline-block;
  padding: 0 1.7rem 0 3.6rem;
  font-size: 1.2rem;
  font-weight: 600;
  line-height: 3.6rem;
  border: 0.2rem solid rgba(255,255,255,0.95);
  border-radius: 2.0rem;
  color: rgba(255,255,255,0.95);
  background: url('../assets/icons/contrast-button.svg') center left 1.1rem no-repeat rgba(255,255,255,0.08);
}

.contrast .expanded {
  position: relative;
  display: inline-block;
  vertical-align: top;
  box-sizing: border-box;
  width: 75%;
  max-width: 32.0rem;
  margin-top: 100%;
  margin-bottom: 2.0rem;
  padding: 2.0rem;
  border-radius: 2.5rem;
  text-align: left;
  font-size: 1.2rem;
  color: rgba(255,255,255,0.95);
  background: #262524;
  box-shadow: 0 0.8rem 0.8rem rgba(0,0,0,0.05),
              0 0.4rem 0.4rem rgba(0,0,0,0.05),
              0 0.2rem 0.2rem rgba(0,0,0,0.05);
}

.contrast .expanded button {
  position: absolute;
  top: 1.2rem;
  right: 1.2rem;
  width: 2.8rem;
  height: 2.8rem;
  border: none;
  border-radius: 50%;
  background: url('../assets/icons/close-white.svg') center no-repeat rgba(255,255,255,0.08);
  border: 0.2rem solid rgba(255,255,255,0.15);
}

.contrast .expanded h3 {
  margin: -0.2rem 0 0;
  font-size: 1.4rem;
  font-weight: 700;
}

.contrast .expanded p {
  margin: 1.3rem 0 1.9rem;
  line-height: 1.4;
}

.contrast .expanded .vertical-toggle:before {
  background: rgba(255,255,255,0.08);
  box-shadow: inset 0 0 0 0.2rem rgba(255,255,255,0.15);
}

.contrast .expanded .vertical-toggle label input:checked {
  background: #fff;
}

.contrast .expanded .vertical-toggle label input:focus-visible {
  box-shadow: 0 0 0 2px #262524, 0 0 0 4px #60ABF0;
}

.contrast .expanded .vertical-toggle label input + span {
  color: rgba(255,255,255,0.35);
}

.contrast .expanded .vertical-toggle label input:checked + span {
  color: rgba(255,255,255,0.95);
}

.contrast .expanded .vertical-toggle label.standard input {
  background: url('../assets/icons/contrast-standard-white.svg') center no-repeat;
}

.contrast .expanded .vertical-toggle label.standard input:checked {
  background: url('../assets/icons/contrast-standard.svg') center no-repeat #C9FE53;
}

.contrast .expanded .vertical-toggle label.standard input:checked + span {
  color: #C9FE53;
}

.contrast .expanded .vertical-toggle label.higher input {
  background: url('../assets/icons/contrast-higher-white.svg') center no-repeat;
}

.contrast .expanded .vertical-toggle label.higher input:checked {
  background: url('../assets/icons/contrast-higher.svg') center no-repeat #fff;
}
</style>