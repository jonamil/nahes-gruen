<template>
  <div
    :class="'modal' + (currentModal ? ' visible' : '')"
    @click.self="hideModal"
    @keydown.esc="hideModal"
  >
    <div ref="window" class="window">
      <button ref="closeButton" title="Schließen" @click="hideModal" />
      <h1>{{ currentModalContent.title }}</h1>
      <template v-if="'paragraphs' in currentModalContent">
        <p
          v-for="(paragraph, index) in currentModalContent.paragraphs"
          :key="index"
          v-html="paragraph"
        />
      </template>
      <template v-else-if="'lists' in currentModalContent">
        <span
          v-for="(list, index) in currentModalContent.lists"
          :key="index"
        >
          <h3>{{ list.heading }}</h3>
          <ol :style="'list-style-type:' + list.style" >
            <li
              v-for="(item, index) in list.items"
              :key="index"
              v-html="parseLinks(item)"
            />
          </ol>
        </span>
      </template>
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
    },
    parseLinks(text) {
      return text.replace(/(?:(?:https?):\/\/|www\.)(?:\([-A-Z0-9+&@#/%=~_|$?!:,.]*\)|[-A-Z0-9+&@#/%=~_|$?!:,.])*(?:\([-A-Z0-9+&@#/%=~_|$?!:,.]*\)|[A-Z0-9+&@#/%=~_|$])/igm, match => {
        return '<a target="_blank" href="' + match + '">' + match + '</a>';
      });
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
  position: fixed;
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
  max-height: 85vh;
  padding: 2.5rem 2.5rem 0;
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

.modal .window > span {
  display: block;
}

.modal .window > :last-child {
  margin-bottom: 2.4rem;
}

.modal .window button {
  position: absolute;
  top: 1.2rem;
  right: 1.2rem;
  width: 2.8rem;
  height: 2.8rem;
  font-size: 0;
  border: none;
  border-radius: 50%;
  background: url('../assets/icons/close.svg') center no-repeat rgba(0,0,0,0.06);
  border: 0.2rem solid rgba(0,0,0,0.1);
}

.modal .window h1 {
  margin: -0.3rem 0 -0.3rem;
  font-size: 1.5rem;
}

.modal .window h3 {
  margin: 1.8rem 0 0;
  font-size: 1.2rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05rem;
  color: rgba(0,0,0,0.65);
}

.modal .window p, .modal .window ol {
  font-size: 1.3rem;
}

.modal .window p {
  margin: 1.7rem 0 -0.1rem;
  line-height: 1.4;
}

.modal .window p img {
  display: block;
  margin: 2.4rem auto;
}

.modal .window a {
  padding-bottom: 0.1rem;
  text-decoration: none;
  box-shadow: inset 0 -0.15rem rgba(0,0,0,0.2);
  color: inherit;
  transition: box-shadow 0.1s ease-in-out;
}

.modal .window a:hover {
  box-shadow: inset 0 -0.15rem rgba(0,0,0,0.65);
}

.modal .window ol {
  margin: 0.9rem 0 0;
  padding-left: 2.6rem;
  line-height: 1.35;
}

.modal .window ol li {
  margin-top: 0.8rem;
  padding-left: 0.3rem;
}

.modal .window ol li a {
  /*overflow-wrap: break-word;
  word-wrap: break-word;*/
  word-break: break-all;
}
</style>