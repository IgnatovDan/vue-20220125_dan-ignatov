<template>
  <h1>With uploader:</h1>
  <div id="app" class="sample container">
    <ui-image-uploader
      :preview="image"
      :uploader="uploadImage"
      @remove="image = undefined"
      @upload="image = $event.image"
      @error="alert($event.message)"
    />
  </div>

  <hr style="margin-top: 10px" />
  <h1>Without uploader:</h1>
  <div id="app" class="sample container">
    <ui-image-uploader :preview="image" @remove="image = undefined" @select="image = $event.image" />
  </div>
</template>

<script>
import UiImageUploader from './components/UiImageUploader';
import { uploadImage } from './ImageService';

export default {
  name: 'App',

  components: { UiImageUploader },

  data() {
    return {
      image: 'https://course-vue.javascript.ru/api/images/1',
      uploadImage,
    };
  },
};
</script>

<style>
@import '~@/assets/styles/_container.css';
</style>
