<script setup lang="ts">
import { onMounted, ref, watch } from 'vue';
import { BehaviorSubject, debounceTime, filter } from 'rxjs';
import { formatLabel } from '@/helpers/bold-text.helper.ts';
import { useI18n } from 'vue-i18n';
import { modulo } from '@/helpers/modulo.helper.ts';

const { t } = useI18n();
const currentSearchValue = defineModel<string>();

const loading = ref<boolean>(false);
const searchSubject = new BehaviorSubject<string>('');
const dropdownOptions = ref<any[]>([]);
const noResult = ref<boolean>(false);
const enterPressed = ref<boolean>(false);
const hoveredDropdownOpenIndex = ref<number>(-1);
const uuid = Math.random();

const props = withDefaults(
  defineProps<{
    options: any[];
    labelProperty: string;
    useCustomDropdownTemplate?: boolean;
    placeholder?: string;
    closeDropdown?: boolean;
    style?: 'default' | 'filter';
    minChars?: number;
    clearValue?: boolean;
    disabled?: boolean;
    translationPrefix?: string;
    defaultValue?: string;
  }>(),
  {
    minChars: 2,
    style: 'default',
    useCustomDropdownTemplate: false,
    disabled: false,
  }
);

const emit = defineEmits(['search', 'select', 'enter', 'clear']);

watch(
  () => props.options,
  () => {
    noResult.value = loading.value && !props.options.length;
    dropdownOptions.value = [...props.options];
    hoveredDropdownOpenIndex.value = -1;
    loading.value = false;
  }
);

watch(
  () => props.closeDropdown,
  () => {
    dropdownOptions.value = [];
    loading.value = false;
    enterPressed.value = false;
    hoveredDropdownOpenIndex.value = -1;
  }
);

watch(
  () => props.clearValue,
  () => {
    dropdownOptions.value = [];
    loading.value = false;
    enterPressed.value = false;
    currentSearchValue.value = '';
    hoveredDropdownOpenIndex.value = -1;
  }
);

// watch(
//   () => props.defaultValue,
//   () => {
//     if (!props.defaultValue) return;
//     currentSearchValue.value = props.defaultValue;
//   }
// );

onMounted(async () => {
  searchSubject
    .pipe(
      debounceTime(400),
      filter(value => {
        const validSearchValue = !!value && value.length >= props.minChars && !enterPressed.value;
        if (!validSearchValue) dropdownOptions.value = [];
        return validSearchValue;
      })
    )
    .subscribe(searchValue => {
      loading.value = searchValue.length >= props.minChars;
      emit('search', searchValue);
    });
});

function input() {
  noResult.value = false;
  searchSubject.next(currentSearchValue.value ?? '');

  if (!currentSearchValue.value?.trim().length) {
    emit('clear');
  }
}

function select(option: any) {
  currentSearchValue.value = props.translationPrefix ? t(`${props.translationPrefix}.${option[props.labelProperty]}`) : option[props.labelProperty];
  dropdownOptions.value = [];
  emit('select', option);
}

function clickOutside(e: any) {
  if ([...e.target.classList].includes(`select-input-${uuid}`)) return;

  if (dropdownOptions.value.length && !dropdownOptions.value.includes(currentSearchValue.value)) {
    emit('clear');
    currentSearchValue.value = '';
  }

  dropdownOptions.value = [];
  noResult.value = false;
}

function enter() {
  if (currentSearchValue.value && currentSearchValue.value.length < 2) return;
  if (hoveredDropdownOpenIndex.value !== -1) {
    select(dropdownOptions.value[hoveredDropdownOpenIndex.value]);
    return;
  }
  enterPressed.value = true;
  emit('enter');
}

function down() {
  hoveredDropdownOpenIndex.value = modulo(hoveredDropdownOpenIndex.value + 1, dropdownOptions.value.length);
}

function up() {
  hoveredDropdownOpenIndex.value = modulo(hoveredDropdownOpenIndex.value - 1, dropdownOptions.value.length);
}
</script>

<template>
  <div class="app-select">
    <div class="input-container" :class="style">
      <input
        type="text"
        v-model="currentSearchValue"
        :placeholder="placeholder"
        :disabled="disabled"
        :class="'select-input-' + uuid"
        @input="input"
        @keyup.enter="enter"
        @keyup.down="down"
        @keyup.up="up"
      />
      <div class="icon" v-if="style === 'default'">
        <div v-if="loading" class="pulse"></div>
        <div v-else><i class="fa-solid fa-search"></i></div>
      </div>
    </div>
    <div
      class="dropdown-results"
      v-click-outside="e => clickOutside(e)"
      :class="{ show: dropdownOptions.length || noResult }"
    >
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
          <p
            v-else-if="currentSearchValue && translationPrefix"
            v-html="formatLabel(t(`${translationPrefix}.${option[labelProperty!]}`), currentSearchValue)"
          ></p>
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
    position: relative;
    display: flex;
    flex: 1;
    min-height: 35px;

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
      width: 20px;
      height: 20px;
      top: 50%;
      transform: translateY(-50%);

      svg {
        transform: scale(1.1) translateY(1px);
      }
    }

    &.filter {
      input {
        padding: 5px;
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
      max-height: 500px;
    }
  }
}
</style>
