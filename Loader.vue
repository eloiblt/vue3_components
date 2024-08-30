<script setup lang="ts">

withDefaults(
  defineProps<{
    size?: string;
    show: boolean;
  }>(),
  {
    size: '40px',
    show: true
  },
);
</script>

<template>
  <transition>
    <div v-if="show" class="app-pulse" :style="{ width: size, height: size }"></div>
  </transition>
</template>

<style lang="scss">
$offset: 187;
$duration: 1.4s;

.app-pulse {
  border-radius: 50%;
  animation: app-pulse 0.8s infinite;
  height: 20px;
  width: 20px;
  background-color: #00549a;
}

@keyframes app-pulse {
  0% {
    transform: scale(0.9);
    box-shadow: 0 0 0 0 rgba(#00549a, 0.7);
  }

  70% {
    transform: scale(1.1);
    box-shadow: 0 0 0 15px rgba(#00549a, 0);
  }

  100% {
    transform: scale(0.9);
    box-shadow: 0 0 0 0 rgba(#00549a, 0);
  }
}

.v-enter-active, .v-leave-active {
  transition: opacity 1s ease;
}

.v-enter-from, .v-leave-to {
  opacity: 0;
}
</style>
