<script setup lang="ts">
import { ref, watch } from 'vue';

const props = withDefaults(
  defineProps<{
    forceState: boolean;
  }>(),
  {}
);

const emit = defineEmits(['change']);

watch(
  () => props.forceState,
  () => {
    checked.value = props.forceState;
  }
);

const checked = ref<boolean>(props.forceState);
</script>

<template>
  <label class="switch">
    <input type="checkbox" v-model="checked" @change="emit('change', checked)" />
  </label>
</template>

<style scoped lang="scss">
input {
  appearance: none;
  width: 58px;
  height: 28px;
  position: relative;
  border-radius: 50px;
  cursor: pointer;
  background-color: var(--text-darker-grey);
  transition: background-color ease 0.3s;
  margin: 0;
  display: flex;

  &:checked {
    background-color: var(--primary-color);

    &:before {
      left: 31px;
    }
  }

  &:before {
    content: '';
    position: absolute;
    width: 23px;
    height: 22px;
    background: #fff;
    left: 2px;
    top: 2px;
    border-radius: 50%;
    color: #fff;
    transition: all cubic-bezier(0.3, 1.5, 0.7, 1) 0.3s;
  }
  &:after {
    content: none;
  }
}
</style>
