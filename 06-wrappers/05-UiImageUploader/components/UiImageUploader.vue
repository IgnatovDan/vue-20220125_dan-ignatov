<template>
  <div class="image-uploader">
    <label
      class="image-uploader__preview"
      :style="{ '--bg-url': bgUrl ? `url('${bgUrl}')` : null }"
      :class="{ 'image-uploader__preview-loading': isLoading }"
    >
      <span class="image-uploader__text">{{ backgroundText }}</span>
      <input
        ref="fileInput"
        type="file"
        v-bind="$attrs"
        accept="image/*"
        class="image-uploader__input"
        @change="handleInputChange"
        @click="handleInputClick"
      />
    </label>
  </div>
</template>

<script>
//
// 'File' component from vuetifyjs: https://vuetifyjs.com/en/components/file-inputs/
// v-model doesn't support 'file': https://stackoverflow.com/questions/64607995/v-model-directives-dont-support-input-type-file
// "c:\fakepath\" + URL.createObjectURL: https://html.spec.whatwg.org/multipage/input.html#fakepath-srsly
// updateImageDisplay: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/file#examples
//

const REMOVE_IMAGE_TEXT = 'Удалить изображение';
const LOAD_IMAGE_TEXT = 'Загрузить изображение';
const LOADING_TEXT = 'Загрузка...';

const strategies = {
  getInitialProps: (uploader, preview) => {
    return {
      strategy: uploader ? strategies.hasUploader : strategies.noUploader,
      isLoading: false,
      bgUrl: preview,
    };
  },

  noUploader: {
    getBackgroundText: (vm) => (vm.bgUrl ? REMOVE_IMAGE_TEXT : LOAD_IMAGE_TEXT),
    handleInputClick: function (vm, event) {
      if (vm.bgUrl || vm.$refs.fileInput.value) {
        event.preventDefault();
        vm.bgUrl = null;
        vm.$refs.fileInput.value = null;
        vm.$emit('remove', { image: null });
      }
    },
    handleInputChange: (vm, event) => {
      if (vm.$refs.fileInput.files.length !== 0) {
        vm.bgUrl = URL.createObjectURL(vm.$refs.fileInput.files[0]);
        vm.$emit('select', vm.$refs.fileInput.files[0]);
      }
    },
  },

  hasUploader: {
    getBackgroundText: (vm) => {
      if (vm.isLoading) {
        return LOADING_TEXT;
      }
      if (vm.bgUrl) {
        return REMOVE_IMAGE_TEXT;
      }
      return LOAD_IMAGE_TEXT;
    },
    handleInputClick: function (vm, event) {
      if (vm.isLoading) {
        event.preventDefault();
        return;
      }
      if (vm.preview || vm.$refs.fileInput.value) {
        event.preventDefault();
        vm.bgUrl = null;
        vm.$refs.fileInput.value = null;
        const isLoading = vm.isLoading;
        vm.isLoading = false;
        if (!isLoading) {
          // rise event after all internal operations are finished
          vm.$emit('remove', { image: null });
        }
      }
    },
    handleInputChange: (vm, event) => {
      vm.uploader(vm.$refs.fileInput.files[0]).then(
        (successResult) => {
          vm.isLoading = false;
          vm.bgUrl = successResult.image;
          vm.$emit('upload', successResult);
        },
        (errorResult) => {
          vm.isLoading = false;
          vm.$refs.fileInput.value = null;
          vm.$emit('error', errorResult);
        },
      );
      vm.isLoading = true;
    },
  },
};

export default {
  name: 'UiImageUploader',

  inheritAttrs: false,

  props: {
    preview: String, // ссылка на отображаемое изображение;
    uploader: Function, // асинхронная функция загрузки изображения.
  },

  emits: [
    'select', // В любом случае после выбора файла порождается событие select с выбранным файлом.
    'upload', // После успешного окончания загрузки порождается событие upload с результатом работы загрузчика.
    'error', // При неуспешной загрузке изображения порождается событие error с ошибкой загрузчика, а компонент возвращается в предыдущее состояние.
    'remove', // При наличии выбранного изображения по клику удаляется изображение и порождается событие remove.
  ],

  data() {
    return strategies.getInitialProps(this.uploader, this.preview);
  },

  computed: {
    backgroundText() {
      return this.strategy.getBackgroundText(this);
    },
  },

  watch: {
    // preview: function(newVal, oldVal) {
    //   Object.assign(this.$data, strategies.getInitialProps(this.hasUploader, newVal));
    // },
    // uploader: function(newVal, oldVal) {
    //   Object.assign(this.$data, strategies.getInitialProps(newVal, this.preview));
    // },
  },

  methods: {
    handleInputChange(e) {
      //
      // v-model doesn't support 'file': https://stackoverflow.com/questions/64607995/v-model-directives-dont-support-input-type-file
      // "c:\fakepath\" + URL.createObjectURL: https://html.spec.whatwg.org/multipage/input.html#fakepath-srsly
      // updateImageDisplay: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/file#examples
      // https://stackoverflow.com/questions/12030686/html-input-file-selection-event-not-firing-upon-selecting-the-same-file
      //

      this.strategy.handleInputChange(this, e);
    },

    handleInputClick(e) {
      this.strategy.handleInputClick(this, e);
    },
  },
};
</script>

<style scoped>
.image-uploader {
}

.image-uploader__input {
  opacity: 0;
  height: 0;
}

.image-uploader__preview {
  --bg-url: var(--default-cover);
  background-size: cover;
  background-position: center;
  background-image: linear-gradient(0deg, rgba(0, 0, 0, 0.4), rgba(0, 0, 0, 0.4)), var(--bg-url);
  border: 2px solid var(--blue-light);
  border-radius: 8px;
  transition: 0.2s border-color;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  max-width: 512px;
  height: 228px;
}

.image-uploader__text {
  color: var(--white);
  font-family: 'Nunito', sans-serif;
  font-weight: 600;
  font-size: 20px;
  line-height: 28px;
}

.image-uploader__preview:hover {
  border-color: var(--blue);
}

.image-uploader__preview.image-uploader__preview-loading {
  cursor: no-drop;
}
</style>
