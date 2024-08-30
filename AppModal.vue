<script setup lang="ts">
import { ref, watch } from 'vue';

const props = withDefaults(
  defineProps<{
    title: string;
    open: boolean;
  }>(),
  {
    open: false,
  }
);

const emit = defineEmits(['close']);
const isOpen = ref<boolean>(props.open);

watch(
  () => props.open,
  () => {
    isOpen.value = props.open;
  }
);

function close() {
  isOpen.value = false;
  emit('close');
}
</script>

<template>
  <Transition name="fade">
    <div class="modal" v-if="isOpen">
      <div class="modal-dialog">
        <div class="header">
          <h2>{{ props.title }}</h2>
          <button type="button" class="btn-no-style" @click="close">
            <i class="fa-solid fa-xmark fa-lg"></i>
          </button>
        </div>
        <div class="content"><slot name="modal-content"></slot></div>
        <div class="footer"><slot name="modal-buttons"></slot></div>
      </div>
    </div>
  </Transition>
</template>

<style scoped lang="scss">
.modal {
  position: fixed;
  inset: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: var(--modal-background);
  z-index: 3;

  .modal-dialog {
    background: var(--card-background);
    min-width: 500px;
    border-radius: 10px;

    .header {
      display: flex;
      background-color: var(--modal-header-bg-color);
      align-items: center;
      padding: 20px 30px;
      border-top-left-radius: 10px;
      border-top-right-radius: 10px;

      h2 {
        font-size: 2rem;
        font-weight: 900;
      }

      button {
        margin-left: auto;
        font-size: 1.5rem;
        padding: 5px;
        color: var(--text-primary);
      }
    }

    .content {
      padding: 20px 30px;
    }

    .footer {
      background-color: var(--modal-header-bg-color);
      display: flex;
      align-items: center;
      gap: 10px;
      padding: 20px 30px;
      border-bottom-left-radius: 10px;
      border-bottom-right-radius: 10px;
    }
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.4s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
