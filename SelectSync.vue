<script setup lang="ts">
import { ref, watch } from 'vue';
import { formatLabel } from '@/helpers/bold-text.helper.ts';
import { useI18n } from 'vue-i18n';

const { t } = useI18n();
const currentSearchValue = defineModel<string>();

const dropdownOptions = ref<any[]>([]);
const noResult = ref<boolean>(false);
const hoveredDropdownOpenIndex = ref<number>(-1);
const uuid = Math.random();

const props = withDefaults(
  defineProps<{
    options: any[];
    labelProperty: string;
    useCustomDropdownTemplate?: boolean;
    placeholder?: string;
    closeDropdown?: boolean;
    minChars?: number;
    clearValue?: boolean;
    disabled?: boolean;
  }>(),
  {
    minChars: 2,
    useCustomDropdownTemplate: false,
    disabled: false,
  }
);

const emit = defineEmits(['select', 'clear']);

watch(
  () => props.closeDropdown,
  () => {
    dropdownOptions.value = [];
    hoveredDropdownOpenIndex.value = -1;
  }
);

watch(
  () => props.clearValue,
  () => {
    dropdownOptions.value = [];
    currentSearchValue.value = '';
    hoveredDropdownOpenIndex.value = -1;
  }
);

function input() {
  noResult.value = false;

  if (!currentSearchValue.value?.trim().length) {
    dropdownOptions.value = [];
    emit('clear');
    return;
  }

  dropdownOptions.value = [...props.options].filter(o =>
    o[props.labelProperty].toLowerCase().includes(currentSearchValue.value?.toLowerCase())
  );

  if (!dropdownOptions.value.length) noResult.value = true;
}

function select(option: any) {
  currentSearchValue.value = option[props.labelProperty];
  dropdownOptions.value = [];
  emit('select', option);
}

function clickOutside(e: any) {
  if ([...e.target.classList].includes(`select-input-${uuid}`)) return;
  dropdownOptions.value = [];
  noResult.value = false;
}

function openDropdown() {
  dropdownOptions.value = [...props.options];
}
</script>

<template>
  <div class="app-select" v-click-outside="e => clickOutside(e)">
    <div class="input-container" @click="openDropdown()">
      <input
        type="text"
        v-model="currentSearchValue"
        :placeholder="placeholder"
        :disabled="disabled"
        :class="'select-input-' + uuid"
        @input="input"
        @keyup.down="down"
        @keyup.up="up"
      />
      <div class="icon">
        <i class="fa-solid fa-caret-down"></i>
      </div>
    </div>
    <div class="dropdown-results" :class="{ show: dropdownOptions.length || noResult }">
      <div v-if="noResult" class="dropdown-option-wrapper disabled">
        <p>{{ t('selectNoResult') }}</p>
      </div>
      <template v-else>
        <div
          class="dropdown-option-wrapper"
          v-for="(option, index) in dropdownOptions"
          :key="option.id"
          @click="select(option)"
          :class="{ hovered: hoveredDropdownOpenIndex === index }"
        >
          <slot v-if="useCustomDropdownTemplate" name="dropdown-option" :option="option"></slot>
          <p v-else-if="currentSearchValue" v-html="formatLabel(option[labelProperty!], currentSearchValue)"></p>
          <p v-else>{{ option[labelProperty!] }}</p>
        </div>
      </template>
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
    min-height: 35px;
    position: relative;
    display: flex;
    flex: 1;

    input {
      flex: 1;
      outline: none;
      caret-color: var(--text-primary);
      width: 100%;
      border: 1px solid var(--disabled);
      border-radius: 6px;
      background: var(--background-secondary);
      padding: 5px 30px 5px 5px;

      &::placeholder {
        color: var(--disabled);
      }

      &:disabled {
        background-color: var(--input-disabled);
        cursor: not-allowed;
      }
    }

    .icon {
      position: absolute;
      right: 5px;
      top: 8px;
      width: 20px;
      height: 20px;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;

      svg {
        transform: scale(1.1);
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
    transition: max-height 0.2s ease-out;
    z-index: 4;
    overflow-y: auto;

    .dropdown-option-wrapper {
      padding: 5px 10px;
      border-radius: 5px;
      display: flex;

      p {
        margin-right: auto;
        white-space: nowrap;
      }

      &:not(.disabled) {
        cursor: pointer;

        &:hover {
          background: var(--background-primary);
        }
      }

      &.hovered {
        background: var(--background-primary);
      }
    }

    &.show {
      padding: 5px;
      max-height: 375px;
    }
  }
}
</style>
