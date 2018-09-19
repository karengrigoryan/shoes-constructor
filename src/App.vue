<template>
  <div id="app" class="container my-5">
    <div class="mt-3 pb-5 constructor">
      <div class="card shadow-card border-0 p-0" style="min-height: 80vh;" v-cloak>
        <div class="card-body d-flex p-0">
          <!-- step #1: model select -->
          <transition name="fade" mode="out-in" appear>
            <!-- loading section -->
            <div v-if="loadingState" class="d-flex flex-column align-items-center justify-content-center w-100">
              <h6 class="font-weight-light text-uppercase" style="color: rgb(115, 115, 115);">Загрузка</h6>

              <div style="width: 10px; height: 10px; background: linear-gradient(to right, #00d2ff, #3a7bd5);" class="rounded-circle animated-pulse mt-3 mx-auto"></div>
            </div>

            <!-- model selection section -->
            <div v-else-if="selectedModel === ''" key="stepSelect" class="position-relative p-4">
              <header class="text-center pt-4">
                <h2 class="font-weight-light mb-1">Выбор модели</h2>
                <p class="small text-muted font-weight-light">Выберите модель чтобы начать сборку</p>
              </header>

              <!-- models grid -->
              <div class="row mt-5 mx-0 px-4">
                <!-- model -->
                <div class="col-sm-6 col-lg-4 col-xl-3 mb-4" v-for="(model, index) in models" :key="`model-${index}`">
                  <a href="#" class="card hover-shadow border-0 no-underline rounded" @click.prevent="selectModel(index)" style="border: 1px solid #e3e3e3; box-shadow: 0 .3rem 3.5rem rgba(0,0,0,.05);">
                    <div class="text-center px-2">
                      <img :src="model.mock_up.sizes.medium" :srcset="`${model.mock_up.sizes.medium} 1x,${model.mock_up.sizes.medium_large} 2x`" :alt="`${model.name} | Ferrone`" class="img-fluid">
                    </div>

                    <h6 class="text-center text-truncate text-white text-uppercase font-weight-light rounded-bottom mb-0 p-3" style="background: linear-gradient(to right, #2b5876, #4e4376); font-size: 15px;">
                      {{ model.name }}</h6>
                  </a>
                </div>
              </div>

              <!-- ENERO copyright -->
              <small class="position-absolute p-2" style="top: 0; right: 0; opacity: .4; font-size: 11px;">
                <a class="text-dark" href="http://www.enero-studio.com" target="_blank">ENERO studio</a> © 2018
              </small>
            </div>

            <!-- constructor section -->
            <div class="w-100" v-else key="stepModel">
              <div class="row no-gutters h-100">
                <!-- controls header -->
                <div class="col-xl position-relative">
                  <header class="d-flex position-absolute p-4" style="top: 0; left: 0; right: 0; z-index: 1">
                    <button type="button" class="font-weight-light small text-uppercase btn btn-dark mr-4" style="height: 40px;" @click="selectedModel = ''">
                      Модели
                    </button>

                    <button type="button" class="font-weight-light small text-uppercase btn btn-success ml-auto" style="height: 40px;" data-toggle="modal" data-target="#modal-order">
                      Заказать
                    </button>
                  </header>

                  <!-- model name and description -->
                  <article class="shoe-title col-xl-5 pt-3 px-3 pb-2">
                    <h4 class="font-weight-light text-dark text-center text-xl-left pb-xl-2">{{
                      models[selectedModel].name }}</h4>

                    <div v-if="models[selectedModel].description" v-html="models[selectedModel].description" class="model-description small font-weight-light"></div>
                  </article>

                  <!-- model preview -->
                  <div class="constructor-preview position-relative my-5 my-xl-0" style="min-height: 400px;">
                    <transition-group name="list" tag="div">
                      <img :src="element.url_medium" :srcset="`${element.url_medium} 1x, ${element.url} 2x`" :alt="models[selectedModel].name" class="img-fluid position-absolute w-100" style="top: 0; right: 0; bottom: 0; left: 0; object-fit: contain;" v-for="(element, index) in selectedModelElements" :key="`model-element-${index}`" v-show="element.visible">
                    </transition-group>
                  </div>
                </div>

                <!-- elements sidebar -->
                <aside class="col-12 col-xl-5 constructor-controls position-relative text-center text-xl-left mt-3 mt-xl-0 pt-4 px-xl-4 pb-xl-4" style="background-color: #ececec;">
                  <div class="d-inline-block d-xl-none bg-dark text-white rounded position-relative px-3" style="top: -30px; margin-top: -5px;">
                    <i class="fas fa-arrows-alt-h fa-lg mt-1"></i>
                  </div>

                  <h3 class="font-weight-light col-12 mb-4 px-0 px-xl-3 d-none d-xl-block">Элементы</h3>

                  <div class="row flex-nowrap flex-xl-wrap position-static mx-0 pt-xl-3 pb-4" style="overflow-x: auto; -webkit-overflow-scrolling: auto;">
                    <!-- element control -->
                    <div class="col-6 col-sm-4 col-md-3 col-lg-2 col-xl-4 position-static constructor-control mb-4 mt-3 mt-xl-0" v-for="(element, index) in models[selectedModel].elements" :key="`element-${index}`">
                      <!-- element toggle (show/hide) -->
                      <a href="#" @click.prevent="toggleElement(index, $event)" class="rounded shadow-sm btn btn-dark d-flex align-items-center justify-content-center shoe-element-toggle position-relative ml-auto p-0" v-if="element.visibility === 'true'">
                        <i class="far fa-eye fa-sm"></i>
                        <i class="far fa-eye-slash fa-sm d-none"></i>
                      </a>

                      <div class="card border-0 shadow-sm position-static" @click="toggleClass('selected', $event)">
                        <img :src="element.mock_up.sizes.medium" alt="element.name" class="mx-auto">

                        <h6 class="text-center small text-muted font-weight-light pt-0 px-2">{{ element.name }}</h6>

                        <!-- variations -->
                        <div class="element-variations-wrapper p-3">
                          <button class="btn btn-link text-dark position-absolute m-2 d-none d-xl-block" style="top: 0; right: 0;">
                            <i class="fas fa-times fa-2x"></i>
                          </button>

                          <div class="row align-items-center element-variations bg-white shadow-sm rounded w-100 mt-2 mx-0 py-2" style="font-size: 10px; cursor: default;" v-for="(material, mIndex) in selectedModelElements[index].materials" @click.stop :key="`material-${mIndex}`">
                            <div class="col-auto pr-0" style="width: 100px">
                              <h6 style="font-size: 13px;" class="font-weight-light text-uppercase text-truncate mb-0">
                                {{ material[0].material }}</h6>
                            </div>

                            <!-- colors -->
                            <div class="col d-flex flex-wrap pl-1">
                              <a href="#" @click.prevent="changeVariation(index, variation.img.sizes.large, variation.img.sizes.medium_large)"
                                      v-for="(variation, index) in material"
                                      :key="`variation-${index}`"
                                      :style="{ backgroundColor: variation.color }" style="width: 20px; height: 20px;"
                                      class="rounded shadow-sm d-inline-block my-1 ml-1">
                              </a>
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>

                    <!-- shoe prophylactic option -->
                    <div class="col-6 col-sm-4 col-md-3 col-lg-2 col-xl-4 constructor-control mb-4 mt-3 mt-xl-0">
                      <div class="card border-0 shadow-sm position-relative">
                        <a href="#" @click.prevent="shoeProphylactic = !shoeProphylactic" :class="{ 'bg-success': shoeProphylactic, 'bg-secondary': !shoeProphylactic }" class="h-100 rounded d-flex pb-3" style="text-decoration: none !important;">
                          <h6 class="text-center text-white d-flex flex-column pt-0 px-2 m-auto small">
                            <transition name="fade" mode="out-in" appear>
                              <span style="font-size: 40px; opacity: .3;" v-if="!shoeProphylactic">+</span>
                              <span style="font-size: 40px; opacity: .3;" v-else>-</span>
                            </transition>

                            Профилактика подошвы
                          </h6>
                        </a>
                      </div>
                    </div>
                  </div>

                  <!-- ENERO copyright -->
                  <small class="position-absolute p-2" style="bottom: 0; right: 0; opacity: .4; font-size: 11px;">
                    <a class="text-dark" href="http://www.enero-studio.com" target="_blank">ENERO studio</a> ©
                    2018
                  </small>
                </aside>
              </div>
            </div>
          </transition>
        </div>
      </div>

      <!-- order modal -->
      <div class="modal fade" id="modal-order" tabindex="-1" role="dialog" aria-labelledby="modal-order-label" aria-hidden="true">
        <div class="modal-dialog modal-lg" role="document">
          <div class="modal-content border-0 shadow-card">
            <!-- heading -->
            <div class="modal-header border-bottom-0">
              <h3 class="modal-title font-weight-light mt-3" id="modal-order-label">Заказать</h3>
            </div>

            <div class="modal-body">
              <!-- form -->
              <form class="pb-3">
                <div class="form-group position-relative pb-4">
                  <input required type="text" class="form-control font-weight-light py-3" placeholder="Имя *" :class="{ 'is-invalid': formOrder.name.errors.length }" v-model="formOrder.name.value" @input="formOrder.name.errors = []">

                  <transition name="fade">
                    <div class="invalid-feedback" v-if="formOrder.name.errors.length">
                      <template v-for="error in formOrder.name.errors">
                        {{ error }}
                      </template>
                    </div>
                  </transition>
                </div>

                <div class="form-group position-relative pb-4">
                  <input required type="text" class="form-control font-weight-light py-3" placeholder="Тел. номер *" :class="{ 'is-invalid': formOrder.tel.errors.length }" v-model="formOrder.tel.value" @input="formOrder.tel.errors = []">

                  <transition name="fade">
                    <div class="invalid-feedback" v-if="formOrder.tel.errors.length">
                      <template v-for="error in formOrder.tel.errors">
                        {{ error }}
                      </template>
                    </div>
                  </transition>
                </div>

                <div class="form-group position-relative pb-4">
                  <input type="email" class="form-control font-weight-light py-3" placeholder="Эл. почта" :class="{ 'is-invalid': formOrder.email.errors.length }" v-model="formOrder.email.value" @input="formOrder.email.errors = []">

                  <transition name="fade">
                    <div class="invalid-feedback" v-if="formOrder.email.errors.length">
                      <template v-for="error in formOrder.email.errors">
                        {{ error }}
                      </template>
                    </div>
                  </transition>
                </div>

                <div class="form-group position-relative pb-4">
                  <input type="text" class="form-control font-weight-light py-3" placeholder="Адрес доставки" v-model="formOrder.address.value">
                </div>

                <div class="form-group position-relative pb-4">
                  <textarea rows="6" type="text" class="form-control font-weight-light py-3" placeholder="Примечания" v-model="formOrder.notes.value" style="resize: none"></textarea>
                </div>

                <div class="form-group position-relative pb-4">
                  <vue-recaptcha ref="recaptcha" :sitekey="recaptchaSitekey" @verify="validateCaptcha" @expired="resetCaptcha"></vue-recaptcha>

                  <transition name="fade">
                    <div class="invalid-feedback" v-if="formOrder.captcha.errors.length">
                      <template v-for="error in formOrder.captcha.errors">
                        {{ error }}
                      </template>
                    </div>
                  </transition>
                </div>

                <!-- buttons -->
                <button type="button" class="btn btn-success position-relative font-weight-light small text-uppercase btn-submit mt-2" @click.prevent="submitForm">
                  <span class="position-absolute d-flex align-items-center justify-content-center" :class="{ 'opacity-0': !fetchingSpinner }">
                      <i class="fas fa-circle-notch fa-spin"></i>
                  </span>

                  <span :class="{ 'opacity-0': fetchingSpinner }">Отправить</span>
                </button>

                <button type="button" class="btn btn-secondary font-weight-light small text-uppercase btn btn-dark mt-2 ml-2" style="height: 40px;" data-dismiss="modal">
                  Закрыть
                </button>
              </form>
            </div>
          </div>
        </div>
      </div>

      <transition name="fade">
        <div style="z-index: 100000; display: flex;" class="bg-danger text-white lead fixed-top shadow-material-2 flex-column align-items-center pb-5 px-3" v-if="formErrorMessage" :class="{ 'opacity-0': !formErrorMessage }" v-cloak>
          <button type="button" class="btn btn-link hover-opacity text-white mt-3 mb-2 ml-auto p-0" @click="formErrorMessage = false">
            <i class="fas fa-lg fa-times ml-auto"></i>
          </button>

          Произошла ошибка. Повторите позже.
        </div>
      </transition>

      <transition name="fade">
        <div style="z-index: 100000; display: flex;" class="bg-success text-white lead fixed-top shadow-material-2 flex-column align-items-center pb-5 px-3" v-if="formSuccessMessage" :class="{ 'opacity-0': !formSuccessMessage }" v-cloak>
          <button type="button" class="btn btn-link hover-opacity text-white mt-3 mb-2 ml-auto p-0" @click="formSuccessMessage = false">
            <i class="fas fa-lg fa-times ml-auto"></i>
          </button>

          Заказ отправлен. Наш сотрудник свяжется с Вами в скором времени.
        </div>
      </transition>
    </div>
  </div>
</template>

<script>
import Axios from 'axios';
import VueRecaptcha from 'vue-recaptcha';

export default {
  name: 'constructor-app',
  components: { VueRecaptcha },
  data() {
    return {
      // fetched data
      models: '',
      modelsFormatted: [],
      selectedModel: '',
      selectedModelElements: [],

      // common options
      shoeProphylactic: false,
      loadingState: true,

      // order form
      urlFormSubmit: 'https://ferrone.ru/wp-json/send-order-form/v1/order',
      recaptchaSitekey: '6Lc0sm0UAAAAAPYIxZ8m9y5fV0pGx7VKH5jAbuaR',
      formErrorMessage: false,
      formSuccessMessage: false,
      formOrder: {
        name: {
          required: true,
          value: '',
          errors: [],
        },
        tel: {
          required: true,
          value: '',
          regex: '^[\\d ()+-]+$',
          errors: [],
        },
        email: {
          value: '',
          regex: '^(([^<>()\\[\\]\\\\.,;:\\s@"]+(\\.[^<>()\\[\\]\\\\.,;:\\s@"]+)*)|(".+"))@((\\[[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\])|(([a-zA-Z\\-0-9]+\\.)+[a-zA-Z]{2,}))$',
          errors: [],
        },
        address: {
          value: '',
        },
        notes: {
          value: '',
        },
        errors: {
          fieldRequired: 'Поле обязательно',
          fieldInvalid: 'Поле заполнено неправильно',
          captchaInvalid: 'Валидация обязательна',
        },
        captcha: {
          validated: '',
          errors: [],
        },
      },
      fetchingSpinner: false,
    };
  },
  computed: {
    // final order
    finalOrder() {
      const f = this.formOrder;
      const finalForm = {};

      // static controls
      finalForm.captcha = f.captcha.validated;
      finalForm.name = f.name.value;
      finalForm.tel = f.tel.value;
      finalForm.prophylactic = this.shoeProphylactic ? 'Да' : 'Нет';
      finalForm.model = this.models[this.selectedModel].name;
      finalForm.elements = [];

      this.selectedModelElements.filter(e => e.visible)
        .forEach((e) => {
          finalForm.elements.push({
            name: e.name,
            url: e.url,
          });
        });

      // conditional controls
      if (f.email.value) {
        finalForm.email = f.email.value;
      }

      if (f.address.value) {
        finalForm.address = f.address.value;
      }

      if (f.notes.value) {
        finalForm.notes = f.notes.value;
      }

      return finalForm;
    },
  },
  methods: {
    // model select
    selectModel(index) {
      // structuring fetched data
      this.selectedModel = index;
      this.selectedModelElements = this.models[index].elements.map((el) => {
        const { variations } = el;
        const materials = variations.map(variation => variation.material);
        const materialsFormatted = [...new Set(materials)];
        const materialsByType = [];

        materialsFormatted.forEach((material) => {
          materialsByType.push(variations.filter(variation => variation.material === material));
        });

        return {
          name: el.name,
          price: el.price,
          url: el.variations[0].img.sizes.large,
          url_medium: el.variations[0].img.sizes.medium_large,
          visible: true,
          materials: materialsByType,
        };
      });

      // pre-loading images
      const tempArr = this.models[index].elements.map(el => el.variations);

      tempArr.forEach(e => e.forEach((variation) => {
        if (window.devicePixelRatio >= 1.5) {
          const img = new Image();
          img.src = variation.img.sizes.large;
        } else {
          const imgMedium = new Image();
          imgMedium.src = variation.img.sizes.medium_large;
        }
      }));
    },

    // changing element variation
    changeVariation(index, newURL, newURLmedium) {
      if (this.selectedModelElements[index].url !== newURL) {
        this.selectedModelElements[index].visible = false;

        setTimeout(() => {
          this.selectedModelElements[index].url = newURL;
          this.selectedModelElements[index].url_medium = newURLmedium;
          this.selectedModelElements[index].visible = true;
        }, 250);
      }
    },

    // toggle element (show/hide)
    toggleElement(index, e) {
      this.selectedModelElements[index].visible = !this.selectedModelElements[index].visible;
      e.target.parentElement.classList.toggle('active');
    },

    // toggling active class on parent element
    toggleClass(classToToggle, e) {
      // vars
      const $card = e.currentTarget;
      const $parentRow = $card.closest('.row');
      const $selectedCard = $parentRow.querySelector(`.${classToToggle}`);
      const isActive = Object.values($card.classList)
        .includes(classToToggle);

        // toggling class on element
      if (isActive) {
        $card.classList.remove(classToToggle);
      } else {
        if ($selectedCard) {
          $selectedCard.classList.remove(classToToggle);
        }

        $card.classList.add(classToToggle);
      }
    },
    // form

    // captcha validation
    validateCaptcha(response) {
      this.formOrder.captcha.errors = [];
      this.formOrder.captcha.validated = response;
    },

    // captcha reset
    resetCaptcha() {
      this.formOrder.captcha.errors.push(this.formOrder.errors.captchaInvalid);
      this.formOrder.captcha.validated = '';
    },

    // validating form
    submitForm() {
      const that = this;

      /* checking if field is empty */
      const isEmpty = str => (!str || /^\s*$/.test(str));

      /* checking if field is matching given regex */
      const isValid = (str, regex) => new RegExp(regex).test(str);

      /* validating each field (fc = form control) */
      Object.values(that.formOrder)
        .forEach((fc) => {
          fc.errors = [];

          /* if not valid */
          if (fc.regex && !isValid(fc.value, fc.regex) && !isEmpty(fc.value)) {
            fc.errors.push(that.formOrder.errors.fieldInvalid);
          } else if (fc.required && isEmpty(fc.value)) {
            fc.errors.push(that.formOrder.errors.fieldRequired);
          }
        });

      /* captcha error */
      if (that.formOrder.captcha.validated === '') this.resetCaptcha();

      /* submitting final form */
      if (!Object.values(that.formOrder)
        .filter(fc => fc.errors.length > 0).length) {
        /* showing spinner until response */
        that.fetchingSpinner = true;

        Axios.post(that.urlFormSubmit, that.finalOrder)
          // success state
          .then(() => {
            that.resetForm();
            that.formSuccessMessage = true;
            that.fetchingSpinner = false;
            setTimeout(() => {
              that.formSuccessMessage = false;
            }, 10000);
          })

        // error state
          .catch(() => {
            that.fetchingSpinner = false;
            this.$refs.recaptcha.reset();
            that.formErrorMessage = true;
          });
      }
    },

    // resetting form
    resetForm() {
      Object.values(this.formOrder)
        .forEach(fc => fc.value = '');
      this.$refs.recaptcha.reset();
      this.formOrder.captcha.validated = '';
    },
  },

  // fetching data from WP
  created() {
    const that = this;
    const pageURL = 'https://ferrone.ru/wp-json/wp/v2/pages/1137';

    Axios.get(pageURL)
      .then((response) => {
        setTimeout(() => {
          that.loadingState = false;
          that.models = response.data.acf.constructor_model;
        }, 4000);
      })
      .catch(() => alert('Error during connection with server. Please try again later.'));
  },
};
</script>

<style lang="scss">
  #app {
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }

  .constructor {
    // constructor styles

    // shoe elements
    .constructor-controls {
      -webkit-overflow-scrolling: touch;
    }

    .constructor-control {
      // card (white wrapper)
      > .card {
        transition: opacity .4s ease-in-out, box-shadow .4s ease-in-out;
        height: 100%;
        min-height: 115px;
        cursor: pointer;

        &.selected:not(.active),
        &:focus,
        &:hover {
          box-shadow: 0 .7rem 1.6rem rgba(#000, .175) !important;
        }

        > img {
          width: 100px;
        }
      }

      // color selector overlay
      .element-variations-wrapper {
        position: absolute;
        right: 0;
        bottom: 100%;
        left: 0;
        z-index: 2;
        pointer-events: none;
        touch-action: none;
        opacity: 0;
        transition: opacity .4s ease-in-out;

        @media (min-width: 1200px) {
          display: flex;
          flex-direction: column;
          align-items: center;
          justify-content: center;
          bottom: 0;
          width: 100%;
          height: 100%;
          background-color: rgba(#fff, .9);
        }
      }

      .element-variations {
        border: 1px solid #efefef;

        a {
          transition: transform .3s;
          &:hover {
            transform: scale(1.3)
          }
        }
      }

      // active (clicked) state
      &.active {
        .shoe-element-toggle {
          border-color: #dc3545 !important;
          background-color: #dc3545 !important;

          i:first-child {
            display: none;
          }
          i:last-child {
            display: block !important;
          }
        }
        > .card {
          opacity: .4;
        }
      }

      // enabled stated
      &:not(.active) .card.selected .element-variations-wrapper {
        opacity: 1 !important;
        pointer-events: auto;
        touch-action: auto;
      }
    }

    // element toggle (show/hide)
    .shoe-element-toggle {
      top: 20px;
      right: -15px;
      z-index: 1;
      width: 30px;
      height: 30px;
      margin-top: -30px;

      > i {
        pointer-events: none;
        touch-action: none;
        transition: background-color .4s ease-in-out, border-color .4s ease-in-out;
      }
    }

    // shoe model preview section
    .constructor-preview {
      pointer-events: none;
      touch-action: none;

      @media (min-width: 1200px) {
        height: 100%;
      }
      img {
        height: 100% !important;
      }
    }

    // shoe title
    .shoe-title {
      position: absolute;
      top: 80px;
      right: 0;
      left: 0;

      @media (min-width: 1200px) {
        top: auto;
        bottom: 0;
        right: auto;
      }
    }

    // description
    .model-description p {
      color: #6c757d;
    }

    // order form
    .modal {
      .form-control {
        border-color: transparent;
        background-color: whitesmoke;
        height: auto;
      }

      .invalid-feedback {
        position: absolute;
        bottom: 0;
      }

      // submit button
      .btn-submit span:first-child {
        position: absolute;
        top: 0;
        right: 0;
        bottom: 0;
        left: 0;
      }
    }

    // utilities
    .hover-shadow {
      transition: box-shadow .4s ease-in-out;
      box-shadow: 0 0.3rem 3.5rem rgba(#000, .15) !important;

      &:focus,
      &:hover {
        box-shadow: 0 0.8rem 4.5rem rgba(#000, .34) !important;
      }
    }

    .animated-pulse {
      animation-name: pulse;
      animation-duration: 1.5s;
      animation-fill-mode: both;
      animation-iteration-count: infinite;

      @keyframes pulse {
        from {
          transform: scale3d(1, 1, 1);
        }
        50% {
          transform: scale3d(1.3, 1.3, 1.3);
        }
        to {
          transform: scale3d(1, 1, 1);
        }
      }
    }

    .shadow-card {
      box-shadow: 0 .125rem 5.25rem rgba(0, 0, 0, .07);
    }

    .no-underline {
      text-decoration: none !important;
    }

    .opacity-0 {
      opacity: 0;
    }

    [v-cloak] {
      display: none !important;
    }

    // vue transition classes
    .fade-enter-active, .fade-leave-active {
      transition: opacity .5s;
    }
    .fade-enter, .fade-leave-to {
      opacity: 0;
    }

    .list-enter-active, .list-leave-active {
      transition: opacity .5s ease-in-out;
    }
    .list-enter, .list-leave-to {
      opacity: 0;
    }
  }
</style>
