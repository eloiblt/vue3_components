<script setup lang="ts">
import { type ColumnConfig } from '@/models/table-data.model';
import { ref, watch } from 'vue';
import { SortOrder } from '@/enums/sort-order.enum.ts';
import { useI18n } from 'vue-i18n';

const props = withDefaults(
  defineProps<{
    config: ColumnConfig;
    widthPx: number;
    headerTranslationPrefix: string;
    resetSort: boolean;
  }>(),
  {}
);

const { t } = useI18n();

const currentSortOrder = ref<SortOrder>(SortOrder.NOSORT);
const emit = defineEmits(['sort']);

watch(
  () => props.resetSort,
  () => {
    if (props.resetSort) {
      currentSortOrder.value = SortOrder.NOSORT;
    }
  }
);

function sort() {
  if (props.config.disableSort) return;

  switch (currentSortOrder.value) {
    case SortOrder.NOSORT:
      currentSortOrder.value = SortOrder.ASC;
      break;
    case SortOrder.ASC:
      currentSortOrder.value = SortOrder.DESC;
      break;
    case SortOrder.DESC:
      currentSortOrder.value = SortOrder.NOSORT;
      break;
  }
  emit('sort', currentSortOrder.value);
}
</script>

<template>
  <th :style="[widthPx ? { 'max-width': `${widthPx}px`, width: `${widthPx}px` } : null]">
    <div class="th-container" @click="sort">
      <slot v-if="config.headerSlot" :name="`${config.property}-header`"></slot>
      <p v-else-if="headerTranslationPrefix && config.property" :class="{ 'can-sort': !config.disableSort }">
        {{ t(`${headerTranslationPrefix}.${config.property}`) }}
      </p>

      <div class="sort-icon" v-if="currentSortOrder === SortOrder.ASC">
        <i class="fa-solid fa-arrow-down"></i>
      </div>
      <div class="sort-icon" v-if="currentSortOrder === SortOrder.DESC">
        <i class="fa-solid fa-arrow-up"></i>
      </div>
    </div>
  </th>
</template>

<style scoped lang="scss">
th {
  top: 0;
  position: sticky;
  z-index: 2;

  .th-container {
    display: flex;
    align-items: center;
    gap: 10px;

    p.can-sort:hover {
      cursor: pointer;
      text-decoration: underline;
      user-select: none;
    }
  }
}
</style>
