<template>
  <div class="root">
    <transition name="fade" mode="out-in">
    <section key="intro" class="introduction" v-if="!introduced">
      <article>
        <p>Bienvenue dans le Timeline-Inventions Challenge !</p>
        <p>
          Le concept est simple : je vous donne le nom d'une invention,
          une trouvaille ou une construction, et vous essayez d'en deviner la date.
        </p>
        <p>Facile non ? Bon allez, une fourchette suffira, disons ... de {{errorMargin}} ans ?</p>
        <p>Bonne chance !</p>
        <p>
          <button v-promise-btn class="btn btn-success" @click="start">Démarrer</button>
        </p>
      </article>
      <article></article>
    </section>
    <section class="game" v-else key="game">
      <article>
        <p>Êtes-vous en mesure de me donner la date de :</p>
        <transition name="fade">
          <p v-if="invention">{{ invention.name }} ?</p>
        </transition>
      </article>
      <article class="slider">
        <vue-slider ref="slider"
                    v-model="response"
                    :fixed="true"
                    :dot-size="15"
                    :height="15"
                    :processDraggable="true"
                    :min="1000"
                    :max="2018">
        </vue-slider>
      </article>
      <article>
        <p>
          <button v-promise-btn="{ loader: '' }" class="btn btn-warning"
                  @click="submit" :disabled="!invention">
            Vérifier
          </button>
        </p>
      </article>
    </section>
    </transition>
    <modal name="result" :adaptive="true" height="auto" @closed="start">
      <div class="result" :success="success">
        <h1>
          <font-awesome-icon v-if="success" icon="check-circle" />
          <font-awesome-icon v-else icon="times-circle" />
        </h1>
        <div v-if="success">
          <p>
            Bien joué !
          </p>
          <p>{{ this.determineMessage }}</p>
        </div>
        <div v-else>{{ this.determineMessage }}</div>
      </div>
    </modal>
  </div>
</template>
<script>
  /**
   * Root vue component containing the app.
   *
   * @author: NikolaLohinski (https://github.com/NikolaLohinski)
   * @date: 05/09/2018
   */
  import vueSlider from 'vue-slider-component';
  const API_URL = `${window.location.protocol}//${window.location.hostname}:5000/api/v0`;
  export default {
    data () {
      const errorMargin = 50;
      return {
        introduced: false,
        errorMargin: errorMargin,
        invention: null,
        response: [1000, 1000 + errorMargin],
        message: {
          VERYCLOSE: 'Presque ! Il manquait trois fois rien !',
          CLOSE: 'Désolé, mais ce n\'est pas tout à fait ça...',
          FAR: 'Aïe, vous étiez très loin !'
        }
      };
    },
    computed: {
      /**
       * Computed variable success to define if the player answered correctly
       * @return {boolean} true if the player answered correctly, false otherwise
       */
      success () {
        return this.invention &&
          (this.response[0] <= this.invention.date && this.invention.date <= this.response[1]);
      },
      /**
       * Determine message to display based on answer
       * @return {string} message to display
       */
      determineMessage () {
        if (!this.invention) return '';
        const date = this.invention.date;
        const distance = Math.min(Math.abs(date - this.response[1]), Math.abs(date - this.response[0]));
        if (this.success) {
          return `${this.invention.name} date très exactement de ${this.invention.date}.`
        } else {
          if (distance < 20) {
            return this.message.VERYCLOSE;
          } else if (distance < 100) {
            return this.message.CLOSE;
          } else {
            return this.message.FAR;
          }
        }
      }
    },
    methods: {
      /**
       * Start function to call to get a random invention and start (or restart) the game
       * @return {Promise} Promise to resolve once we get the request
       */
      start () {
        this.introduced = true;
        return new Promise((resolve, reject) => {
          this.$http.get(API_URL + '/inventions/random').then((response) => {
            this.invention = null;
            setTimeout(() => {
              this.invention = response.body.result.invention;
              resolve();
            }, 200);
          }, reject);
        });
      },
      /**
       * Show result modal stating if the answer is correct or not
       * @return {Promise} Promise to resolve by showing the modal
       */
      submit () {
        return new Promise((resolve) => {
          resolve();
          this.$modal.show('result');
        });
      }
    },
    components: {
      vueSlider
    }
  };
</script>
<style lang="sass" type="text/scss" rel="stylesheet/scss" scoped>
  @import '../scss/main';
  .root {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    font-family: $font-family;
    color: $font-color;
    background-color: $background-color;
    section {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      width: 90vw;
      max-width: $max-width;
      text-align: center;
      article.slider {
        margin: 45px auto;
        width: 80vw;
        max-width: $max-width;
      }
    }
    .result {
      text-align: center;
      padding: 30px;
      h1 {
        font-size: 75px;
        color: $failure-color;
      }
      &[success] h1 {
        color: $success-color;
      }
    }
  }
  .fade-enter-active,
  .fade-leave-active {
    transition: opacity 200ms;
  }
  .fade-enter,
  .fade-leave-to {
    opacity: 0;
  }
</style>
