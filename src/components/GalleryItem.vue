<template>
  <li class="gallery-item">
    <div class="img-container">
      <img class="img" :src="data.isUrlCorrupted ? placeholderImage : data.url" :alt="data.label"
        :onerror="handleErrLoadingImage" />
    </div>
    <div class="label">
      {{ data.label }}
    </div>
  </li>
</template>

<script lang="ts">
  import { defineComponent } from "vue";
  import type { PropType } from "vue";

  import placeholderImage from "@/assets/no-image.png";
  import type { SingleImageData } from "@/types";

  export default defineComponent({
    props: {
      data: { type: Object as PropType<SingleImageData> },
    },
    emits: ["onImageUrlCorrupted"],
    setup(props, { emit }) {
      const handleErrLoadingImage = () => {
        emit("onImageUrlCorrupted", props.data?.id);
      }

      return {
        props,
        placeholderImage,
        handleErrLoadingImage,
      };
    },
  });
</script>

<style scoped lang="scss">
  .gallery-item {
    margin: 6px;
    width: 160px;
    height: 140px;
    display: flex;
    flex-direction: column;
    justify-content: flex-end;

    .img-container {
      margin-bottom: 6px;
      width: 100%;
      display: flex;
      justify-content: center;
      border-radius: 6px;
      align-items: center;
      overflow: hidden;

      .img {
        width: 100%;
        height: 100%;
        object-fit: cover;
      }
    }

    .label {
      width: 100%;
      font-size: 12px;
      font-family: serif;
      color: #000;
      overflow: hidden;
      white-space: nowrap;
      text-overflow: ellipsis;
    }
  }
</style>