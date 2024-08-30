<script setup lang="ts">
import { ref, watch } from 'vue';

const inputValue = ref<string>('');

const selectedOptions = ref<any[]>([]);
const openDropdown = ref<boolean>(false);
const uuid = Math.random();

const props = withDefaults(
  defineProps<{
    options: any[];
    labelProperty: string;
    clearValue?: boolean;
    initialValue?: any[];
  }>(),
  {}
);

const emit = defineEmits(['change']);

watch(
  () => props.initialValue,
  () => {
    selectedOptions.value = props.initialValue || [];
    inputValue.value = selectedOptions.value.map(o => o[props.labelProperty]).join(', ');
  },
  { immediate: true }
);

watch(
  () => props.clearValue,
  () => {
    inputValue.value = '';
  }
);

function select(option: any) {
  if (selectedOptions.value.map(c => c[props.labelProperty]).includes(option[props.labelProperty])) {
    selectedOptions.value = selectedOptions.value.filter(o => o[props.labelProperty] !== option[props.labelProperty]);
  } else {
    selectedOptions.value = [...selectedOptions.value, option];
  }

  inputValue.value = selectedOptions.value.map(o => o[props.labelProperty]).join(', ');
  emit('change', selectedOptions.value);
}

function clickOutside(e: any) {
  if ([...e.target.classList].includes(`select-input-${uuid}`)) return;
  openDropdown.value = false;
}

function clear() {
  selectedOptions.value = [];
  inputValue.value = '';
  emit('change', []);
}
</script>

<template>
  <div class="app-select" v-click-outside="e => clickOutside(e)">
    <div class="input-container">
      <div class="select-input" :class="'select-input-' + uuid" @click="openDropdown = true">{{ inputValue }}</div>
      <div class="icon">
        <button type="button" class="btn-no-style" v-if="inputValue" @click="clear">
          <i class="fa-solid fa-xmark"></i>
        </button>
        <button type="button" class="btn-no-style" @click="openDropdown = true" v-else>
          <i class="fa-solid fa-caret-down"></i>
        </button>
      </div>
    </div>
    <div class="dropdown-results" :class="{ show: openDropdown }">
      <div class="dropdown-option" v-for="(option, index) in options" :key="index" @click="select(option)">
        <input type="checkbox" :checked="selectedOptions.includes(option)" />
        <p>{{ option[labelProperty!] }}</p>
      </div>
    </div>
  </div>
</template>

<style scoped lang="scss">
.app-select {
  display: flex;
  flex: 1;
  flex-direction: column;
  position: relative;
  height: 100%;

  .input-container {
    position: relative;
    display: flex;
    flex: 1;

    .select-input {
      flex: 1;
      width: 0;
      border: 1px solid var(--disabled);
      border-radius: 6px;
      background: var(--background-secondary);
      padding: 8px 30px 5px 5px;

      &::placeholder {
        color: var(--disabled);
      }
    }

    .icon {
      position: absolute;
      right: 5px;
      top: 0;
      bottom: 0;
      margin-top: auto;
      margin-bottom: auto;
      width: 20px;
      height: 20px;
      display: flex;
      align-items: center;
      justify-content: center;

      button {
        cursor: pointer;

        svg {
          transform: scale(1.1);
        }
      }
    }
  }

  .dropdown-results {
    background: white;
    position: absolute;
    top: 100%;
    left: 0;
    min-width: calc(100% - 10px);
    border-radius: 5px;
    box-shadow: 0 0 10px -7px var(--card-shadow);
    display: flex;
    flex-direction: column;
    max-height: 0;
    overflow: hidden;
    z-index: 4;
    transition: max-height 0.2s ease-out;
    overflow-y: auto;

    .dropdown-option {
      padding: 5px 10px;
      border-radius: 5px;
      display: flex;
      gap: 10px;
      cursor: pointer;

      input[type='checkbox'] {
        transform: scale(1.3);
        margin: 0;
      }

      p {
        margin-right: auto;
        white-space: nowrap;
      }

      &:hover {
        background: var(--background-primary);
      }
    }

    &.show {
      padding: 5px;
      max-height: 300px;
    }
  }
}
</style>
