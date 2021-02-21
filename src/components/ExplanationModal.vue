<template>
  <div
    :class="'modal' + (currentModal ? ' visible' : '')"
    @click.self="hideModal"
    @keydown.esc="hideModal"
  >
    <div ref="window" class="window">
      <button ref="closeButton" title="Schließen" @click="hideModal" />
      <h3 v-if="currentModalContent.key !== 'sources'">Erläuterung</h3>
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
        this.currentModalContent.key = currentModal;
        this.$refs.window.scrollTop = 0;
      } else if (currentModal !== false) {
        this.hideModal();
      }
    }
  },

  mounted () {
    document.addEventListener('keydown', event => {
      if (this.currentModal) {
        if (event.keyCode === 27) {
          this.hideModal();
        } else if (event.keyCode === 9) {
          this.$refs.closeButton.focus();
          event.preventDefault();
        }
      }
    });

    this.$refs.window.addEventListener('click', event => {
      if (event.target.localName === 'sup') {
        this.$emit('update:currentModal', 'sources');
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
  background: rgba(0,0,0,0.625);
  transition: visibility 0.1s ease-in-out, opacity 0.1s ease-in-out;
  z-index: 300;
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
  max-width: 60.0rem;
  max-height: 75vh;
  padding: 2.5rem;
  border-radius: 2.5rem;
  background: #F8F3EB;
  box-shadow: 0 0.8rem 0.8rem rgba(0, 0, 0, 0.03),
              0 0.4rem 0.4rem rgba(0, 0, 0, 0.03),
              0 0.2rem 0.2rem rgba(0, 0, 0, 0.03);
  transform: translateX(-50%) translateY(-50%) scale(0.98);
  transition: transform 0.1s ease-in-out;
}

.modal.visible .window {
  transform: translateX(-50%) translateY(-50%) scale(1);
}

.modal .window button {
  position: absolute;
  top: 1.2rem;
  right: 1.2rem;
  width: 2.8rem;
  height: 2.8rem;
  border: none;
  border-radius: 50%;
  background: url('../assets/icons/close.svg') center no-repeat rgba(0,0,0,0.06);
  border: 0.2rem solid rgba(0,0,0,0.1);
}

.modal .window h3 {
  margin: -0.2rem 0 0;
  font-size: 1.2rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05rem;
  color: rgba(0,0,0,0.3);
}

.modal .window h1 {
  margin: -0.2rem 0 -0.3rem;
  font-size: 1.6rem;
}

.modal .window h3 + h1 {
  margin-top: 0.2rem;
}

.modal .window p {
  margin: 1.7rem 0 -0.1rem;
  font-size: 1.3rem;
  line-height: 1.4;
}

.modal .window p + p {
  /*margin-top: 1.0rem;*/
}
</style>