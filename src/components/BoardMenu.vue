<script setup lang="ts">
import { Popup as KPopup } from "@progress/kendo-vue-popup";
import { Button as KButton } from "@progress/kendo-vue-buttons";
import { ref } from "vue";
import type { Board } from "@/types";
import { onClickOutside } from "@vueuse/core";
import { useMutation } from "@vue/apollo-composable";
import attachImageToBoardMutation from "@/graphql/mutations/attachImageToBoard.mutation.gql";
import { useAlerts } from "@/stores/alerts";

const alerts = useAlerts();

const props = defineProps<{
  board: Board;
}>();

const show = ref(false);
const menu = ref(null);
onClickOutside(menu, () => setTimeout(() => (show.value = false), 2));

const emit = defineEmits<{
  (e: "deleteBoard", payload: null): void;
  (e: "imageUpload", payload: { id: string }): void;
}>();

const {
  mutate: attachImageToBoard,
  onError: errorAttachingImage,
  onDone: onImageAttached,
  loading: imageLoading,
} = useMutation(attachImageToBoardMutation);
errorAttachingImage((error) => {
  console.log(error);
  alerts.error("Error setting board image");
});
onImageAttached((result) => {
  emit("imageUpload", result.data.boardUpdate.image);
});
</script>
<template>
  <div>
    <KButton
      icon="folder"
      theme-color="primary"
      fillMode="outline"
      @click="show = !show"
      ref="button"
      >Show Menu</KButton
    >
    <KPopup
      :anchor="'button'"
      :anchor-align="{
        vertical: 'bottom',
        horizontal: 'right',
      }"
      :popup-align="{
        horizontal: 'right',
        vertical: 'top',
      }"
      :show="show"
    >
      <div class="p-5" ref="menu">
        <ul>
          <li class="text-red-500 whitespace-nowrap">
            <button @click="$emit('deleteBoard', null)">
              <span class="k-icon k-i-delete"></span>
              Delete Board
            </button>
          </li>
          <li>
            <strong>Board Image</strong>
            <AppImageDropzone
              class="aspect-video w-56"
              :image="board.image?.downloadUrl"
              :loading="imageLoading"
              @upload="
                attachImageToBoard({
                  id: board.id,
                  imageId: $event.id,
                })
              "
            />
          </li>
        </ul>
      </div>
    </KPopup>
  </div>
</template>
<style scoped>
ul li {
  @apply p-2;
  border-bottom: 1px solid #eee;
}
ul li:last-of-type {
  border-bottom: none;
}
</style>
