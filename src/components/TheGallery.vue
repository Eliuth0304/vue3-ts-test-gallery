<template>
  <div class="gallery-layout">
    <ul v-if="dataCollection.length" class="list-container">
      <gallery-item v-for="(image, idx) in dataCollection" :key="image.id" :data="image"
        @onImageUrlCorrupted="handleImageUrlCorrupted" />
    </ul>
    <div v-else class="placeholder">
      No Images Available
    </div>
  </div>
</template>

<script lang="ts">
  import { defineComponent, onMounted, watch, ref } from "vue";
  import { v4 as uuidv4 } from "uuid";

  import GalleryItem from "./GalleryItem.vue";
  import type { SingleImageData } from "@/types";

  export const DATA_S3_URL = "https://s3.eu-west-2.amazonaws.com/assets-test.fast-thinking.co.uk/recruitment/data.json";
  export const FETCH_TRY_LIMIT = 10;

  export default defineComponent({
    components: {
      GalleryItem,
    },

    setup() {
      let dataCollection = ref<SingleImageData[]>([]);
      let fetchedCount: number = 0;
      let timerInstance: ReturnType<typeof setTimeout> = null; // instance of timer for fetching corrupted images

      const fetchImagesDataFromS3 = async (): void => {
        fetchedCount++;
        try {
          let response = await fetch(DATA_S3_URL);

          if (!response?.ok) {
            console.log(`Status Code: ${response?.status}`);
            return;
          }

          const responseData = await response.json();

          if (fetchedCount === 1) {
            dataCollection.value = responseData.map((item: SingleImageData): SingleImageData => ({
              ...item,
              id: uuidv4(),
            }));
          }
          dataCollection.value = dataCollection.value.map((item: SingleImageData): SingleImageData => ({
            ...item,
            isUrlCorrupted: false,
          }));
        } catch (e) {
          console.error(e);
        }
      }

      const refetchImages = async (): void => {
        timerInstance && clearTimeout(timerInstance);
        if (fetchedCount > FETCH_TRY_LIMIT) {
          return;
        }
        timerInstance = setTimeout(
          await fetchImagesDataFromS3,
          Math.pow(2, fetchedCount) * 1000
        );
      };

      onMounted(fetchImagesDataFromS3);

      watch(dataCollection, refetchImages, { deep: true });

      const handleImageUrlCorrupted = (id: string): void => {
        const corruptedImageIndex: number = dataCollection.value.findIndex((item) => item.id === id);

        dataCollection.value[corruptedImageIndex] = {
          ...dataCollection.value[corruptedImageIndex],
          isUrlCorrupted: true,
        };
      }

      return {
        dataCollection,
        handleImageUrlCorrupted,
      };
    },
  });
</script>

<style scoped lang="scss">
  .gallery-layout {
    display: flex;
    justify-content: center;

    .list-container {
      margin: 0;
      padding: 0;
      width: 100%;
      display: flex;
      justify-content: center;
      flex-wrap: wrap;
      list-style: none;
    }

    .placeholder {
      margin: auto;
      color: gray;
      font-size: 24px;
    }
  }
</style>