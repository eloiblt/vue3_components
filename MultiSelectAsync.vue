<script setup lang="ts">
import { onMounted, ref, watch } from 'vue';
import { BehaviorSubject, debounceTime, filter } from 'rxjs';

const currentSearchValue = defineModel<string>();

const searchSubject = new BehaviorSubject<string>('');
const dropdownOptions = ref<any[]>([]);
const selectedOptions = ref<any[]>([]);
const uuid = Math.random();
const showDropdown = ref<boolean>(false);

const props = withDefaults(
  defineProps<{
    options: any[];
    labelProperty: string;
    minChars?: number;
    clearValue?: boolean;
    disabled?: boolean;
    translationPrefix?: string;
  }>(),
  {
    minChars: 2,
    disabled: false,
  }
);

const emit = defineEmits(['select', 'search', 'clear']);

onMounted(async () => {
  searchSubject
    .pipe(
      debounceTime(400),
      filter(value => {
        const validSearchValue = !!value && value.length >= props.minChars;
        if (!validSearchValue) dropdownOptions.value = [];
        return validSearchValue;
      })
    )
    .subscribe(searchValue => {
      emit('search', searchValue);
    });
});

watch(
  () => props.options,
  () => {
    dropdownOptions.value = [...props.options];
    showDropdown.value = !!dropdownOptions.value.length;
  }
);

watch(
  () => props.clearValue,
  () => {
    clear();
  }
);

function emitSearch() {
  currentSearchValue.value = selectedOptions.value.map(o => o[props.labelProperty]).join(', ');
  emit(
    'select',
    selectedOptions.value.map(s => s[props.labelProperty])
  );
}

function select(option: any) {
  if (selectedOptions.value.map(c => c[props.labelProperty]).includes(option[props.labelProperty])) {
    selectedOptions.value = selectedOptions.value.filter(o => o[props.labelProperty] !== option[props.labelProperty]);
  } else {
    selectedOptions.value = [...selectedOptions.value, option];
  }
}

function input() {
  searchSubject.next(currentSearchValue.value ?? '');
}

function clickOutside(e: any) {
  if ([...e.target.classList].includes(`${uuid}`)) return;
  if (props.disabled) return;

  showDropdown.value = false;

  if (selectedOptions.value.length) {
    emitSearch();
  } else {
    dropdownOptions.value = [];
    currentSearchValue.value = '';
    emit('clear');
  }
}

function clear() {
  selectedOptions.value = [];
  dropdownOptions.value = [];
  currentSearchValue.value = '';

  emit('clear');
}

function focus() {
  if (!dropdownOptions.value.length) return;
  showDropdown.value = !showDropdown.value;

  if (!showDropdown.value && selectedOptions.value.length) {
    emitSearch();
  }
}
</script>

<template>
  <div class="app-select" v-click-outside="e => clickOutside(e)">
    <div class="input-container">
      <input type="text" v-model="currentSearchValue" :disabled="disabled" :class="`${uuid}`" @input="input" />
      <div class="icon">
        <button type="button" v-if="dropdownOptions.length" class="btn-no-style" @click="focus">
          <i class="fa-solid fa-caret-down"></i>
        </button>
        <button type="button" v-else-if="currentSearchValue?.length" class="btn-no-style" @click="clear">
          <i class="fa-solid fa-xmark"></i>
        </button>
      </div>
    </div>
    <div class="dropdown-results" :class="{ show: showDropdown }">
      <div class="dropdown-option" v-for="(option, index) in dropdownOptions" :key="index" @click="select(option)">
        <input type="checkbox" :checked="selectedOptions.includes(option)" />
        <p v-if="translationPrefix" v-html="t(`${translationPrefix}.${option[labelProperty!]}`)"></p>
        <p v-else>{{ option[labelProperty!] }}</p>
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
    max-height: 0 !important;
    overflow: hidden;
    z-index: 4;
    overflow-y: auto;

    .dropdown-option {
      padding: 5px 10px;
      border-radius: 5px;
      display: flex;
      gap: 10px;
      cursor: pointer;

      input[type='checkbox'] {
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
      max-height: 300px !important;
    }
  }
}
</style>
