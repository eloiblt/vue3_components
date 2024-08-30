<script setup lang="ts">
import { onMounted, ref, watch } from 'vue';
import { useI18n } from 'vue-i18n';
import type { AxiosObservable } from 'axios-observable';
import { firstValueFrom } from 'rxjs';
import {
  type BaseSearchQuery,
  type CentralOfficeFiltersSearchQuery,
  type SearchQueryFilter,
} from '@/models/search-query.model.ts';
import { type FilterConfig } from '@/models/table-data.model.ts';
import SelectAsync from '@/components/shared/SelectAsync.vue';
import MultiSelectAsync from '@/components/shared/MultiSelectAsync.vue';

const { t } = useI18n();
const emit = defineEmits(['search', 'add-filter', 'remove-filter']);
const dropdownResults = ref<Map<string, Array<{ label: string }>>>(new Map<string, Array<{ label: string }>>());
const clearSelects = ref<boolean>(false);

const props = withDefaults(
  defineProps<{
    filtersConfig: FilterConfig[];
    apiCall: (query) => AxiosObservable<string[]>;
    searchQuery: BaseSearchQuery;
    translationPrefix: string;
  }>(),
  {}
);

onMounted(() => {
  emptyDropdowns();
});

watch(
  () => props.searchQuery,
  () => {
    emptyDropdowns();
    if (!props.searchQuery.filters?.length) {
      clearSelects.value = true;
    }
  }
);

watch(
  () => clearSelects.value,
  () => {
    setTimeout(() => (clearSelects.value = false), 10);
  }
);

function emptyDropdowns() {
  for (const filter of props.filtersConfig) {
    dropdownResults.value.set(filter.property, []);
  }
}

async function searchDropdown(searchTerm: string, filter: FilterConfig) {
  searchTerm = searchTerm.trim();
  if (!searchTerm) return;

  const filtersQuery = {
    property: filter.property,
    searchTerm,
    searchQuery: props.searchQuery,
  } satisfies CentralOfficeFiltersSearchQuery;

  const response = (await firstValueFrom(props.apiCall(filtersQuery))).data;
  dropdownResults.value.set(
    filter.property,
    response.map(r => {
      return { label: r };
    })
  );
}

function selectFromDropdown(filter: FilterConfig, values: string[]) {
  emit('add-filter', { property: filter.property, values } satisfies SearchQueryFilter);
}
</script>

<template>
  <div class="filters-container">
    <div class="filter" v-for="(filter, index) in filtersConfig" :key="index" :class="{ grow: !filter.widthPx }">
      <p>{{ t(`${translationPrefix}.${filter.property}`) }}</p>

      <div class="filter-select-wrapper" :style="{ width: filter.widthPx + 'px' }">
        <SelectAsync
          v-if="!filter.multiple"
          :options="dropdownResults.get(filter.property)!"
          :label-property="'label'"
          :style="'filter'"
          :min-chars="1"
          :clear-value="clearSelects"
          :translation-prefix="filter.translationPrefix"
          :placeholder="filter.placeholder"
          @clear="emit('remove-filter', filter.property)"
          @select="selectFromDropdown(filter, [$event.label])"
          @search="searchDropdown($event, filter)"
        >
        </SelectAsync>

        <MultiSelectAsync
          v-if="filter.multiple"
          :options="dropdownResults.get(filter.property)!"
          :label-property="'label'"
          :clear-value="clearSelects"
          :translation-prefix="filter.translationPrefix"
          @clear="emit('remove-filter', filter.property)"
          @select="selectFromDropdown(filter, $event)"
          @search="searchDropdown($event, filter)"
        ></MultiSelectAsync>
      </div>
    </div>
  </div>
</template>

<style scoped lang="scss">
.filters-container {
  display: flex;
  align-items: center;
  background: #f4f4f4;
  border-radius: 5px;
  gap: 15px;
  max-height: 0;
  opacity: 0;
  transition: all 0.2s;
  padding: 0;

  p {
    font-weight: bold;
  }

  .filter {
    display: flex;
    align-items: center;
    height: 35px;
    gap: 10px;

    .filter-select-wrapper {
      display: flex;
      flex: 1;
      height: 30px;
    }

    &.grow {
      flex: 1;
    }
  }
  
  * {
    max-height: 0;
    opacity: 0;
  }

  &.show {
    max-height: 100px;
    opacity: 1;
    padding: 10px;
    
    * {
      max-height: unset;
      opacity: 1;
    }
  }
}
</style>
