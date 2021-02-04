<template>
  <div
    :class="'modal' + (currentModal ? ' visible' : '')"
    @click.self="hideModal"
    @keydown.esc="hideModal"
  >
    <div ref="window" class="window">
      <button title="Schließen" @click="hideModal" />
      <h3>Erläuterung</h3>
      <h1>{{ currentModalContent.title }}</h1>
      <p
        v-for="(paragraph, index) in currentModalContent.paragraphs"
        :key="index"
        v-html="paragraph"
      />
    </div>
  </div>
</template>

<script>
import modalContents from '../data/modal-contents.json';

export default {
  name: 'ExplanationModal',

  props: {
    currentModal: {
      type: [String, Boolean],
      default: false
    }
  },

  data () {
    return {
      modalContents,
      currentModalContent: modalContents[Object.keys(modalContents)[0]]
    }
  },

  methods: {
    hideModal() {
      this.$emit('update:currentModal', false);
    }
  },

  watch: {
    currentModal: function(currentModal) {
      if (currentModal in this.modalContents) {
        this.currentModalContent = this.modalContents[currentModal];
        this.$refs.window.scrollTop = 0;
      } else if (currentModal !== false) {
        this.hideModal();
      }
    }
  },

  mounted () {
    document.addEventListener('keydown', event => {
      if (this.currentModal && event.keyCode === 27) {
        this.hideModal();
      }
    });
  }
}
</script>

<style>
.modal {
  visibility: hidden;
  opacity: 0;
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background: rgba(0,0,0,0.6);
  transition: visibility 0.1s ease-in-out, opacity 0.1s ease-in-out;
}

.modal.visible {
  visibility: visible;
  opacity: 1;
}

.modal .window {
  position: relative;
  overflow-y: auto;
  box-sizing: border-box;
  top: 50%;
  left: 50%;
  width: 75vw;
  max-width: 600rem;
  max-height: 75vh;
  /*margin: -200rem 0 0 -300rem;*/
  padding: 25rem;
  border-radius: 25rem;
  background: #F8F3EB;
  box-shadow: 0rem 8rem 8rem rgba(0, 0, 0, 0.03),
              0rem 4rem 4rem rgba(0, 0, 0, 0.03),
              0rem 2rem 2rem rgba(0, 0, 0, 0.03);
  transform: translateX(-50%) translateY(-50%) scale(0.98);
  transition: transform 0.1s ease-in-out;
}

.modal.visible .window {
  transform: translateX(-50%) translateY(-50%) scale(1);
}

.modal .window button {
  -webkit-appearance: none;
  appearance: none;
  cursor: pointer;
  position: absolute;
  box-sizing: border-box;
  top: 12rem;
  right: 12rem;
  width: 28rem;
  height: 28rem;
  border: none;
  border-radius: 50%;
  background: url('../assets/icons/close.svg') center no-repeat rgba(0,0,0,0.06);
  border: 2rem solid rgba(0,0,0,0.1);
}

.modal .window button:focus {
  outline: none;
}

.modal .window button:focus-visible {
  box-shadow: 0 0 0 2px #FAF6F0, 0 0 0 4px rgba(0,0,0,0.1);
}

.modal .window h3 {
  margin: -1rem 0 0;
  font-size: 12rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5rem;
  color: rgba(0,0,0,0.3);
}

.modal .window h1 {
  margin: 3rem 0 -2rem;
  font-size: 18rem;
  /*font-weight: 700;*/
}

.modal .window p {
  margin: 16rem 0 0;
  font-size: 14rem;
  line-height: 1.35;
}

.modal .window p + p {
  /*margin-top: 10rem;*/
}
</style>