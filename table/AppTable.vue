<script setup lang="ts">
import TableRow from './TableRow.vue';
import { type ColumnConfig, type TableConfig } from '@/models/table-data.model.ts';
import TableHeader from './TableHeader.vue';
import { ref, watch } from 'vue';
import { SortOrder } from '@/enums/sort-order.enum.ts';
import { ColsTypeEnum } from '@/enums/cols-type.enum.ts';
import { useI18n } from 'vue-i18n';
import TablePaginate from './TablePaginate.vue';

const props = withDefaults(
  defineProps<{
    totalCount?: number; // not required if addPagination = false
    data: any[];
    config: TableConfig;
    fixedFirstCol?: boolean;
    fixedLastCol?: boolean;
    addPagination?: boolean;
    emitOnSort?: boolean;
    resetPagination?: boolean;
    maxHeightPx?: number;
    rowClickable?: boolean;
  }>(),
  {
    fixedFirstCol: false,
    fixedLastCol: false,
    addPagination: true,
    emitOnSort: false,
    rowClickable: true,
  }
);

const { t } = useI18n();

const tableContainer = ref();
const widthsPx = ref<number[]>([]);
const sortedColum = ref<ColumnConfig>();
const lines = ref<any[]>([]);
const pagesCount = ref<number>(0);
const resetPaginationPage = ref<boolean>(false);
const possiblePageLength = ref<number[]>([10, 25, 50, 100]);
const currentPage = ref<number>(1);
const pageSize = ref<number>(10);

const emit = defineEmits(['pageSize', 'page', 'sort', 'rowClick']);

watch(
  () => props.data,
  () => {
    lines.value = [...props.data];
    if (lines.value.length) setWidthsPx();
  },
  { immediate: true }
);

function setWidthsPx() {
  (function trySetWidthsPx() {
    if (!tableContainer.value?.offsetWidth) {
      setTimeout(trySetWidthsPx, 100);
      return;
    }

    props.config.columnsConfig.forEach((c, i) => {
      widthsPx.value[i] = ((c.widthPercent ?? 0) / 100) * tableContainer.value.offsetWidth;
    });
  })();
}

watch(
  () => props.resetPagination,
  () => {
    resetPaginationPage.value = true;
    setTimeout(() => (resetPaginationPage.value = false), 500);
  }
);

watch(
  () => lines.value,
  () => {
    if (!props.addPagination || !props.totalCount) return;
    pagesCount.value = Math.ceil(props.totalCount / pageSize.value);
  },
  { immediate: true }
);

function sort(columnConfig: ColumnConfig, order: SortOrder) {
  sortedColum.value = columnConfig;

  if (props.emitOnSort) {
    emit('sort', { columnConfig, order });
    return;
  }

  if (order === SortOrder.NOSORT) {
    lines.value = [...props.data];
    return;
  }

  const property = columnConfig.property!;
  if (columnConfig.type === ColsTypeEnum.text) {
    if (order === SortOrder.ASC) {
      lines.value.sort((a, b) => {
        if (a[property] === null || a[property].trim() === '') return 1;
        if (b[property] === null || b[property].trim() === '') return -1;
        return a[property]?.localeCompare(b[property]);
      });
    } else {
      lines.value.sort((a, b) => {
        if (a[property] === null || a[property].trim() === '') return 1;
        if (b[property] === null || b[property].trim() === '') return -1;
        return b[property]?.localeCompare(a[property]);
      });
    }
  }

  if (columnConfig.type === ColsTypeEnum.decimal) {
    if (order === SortOrder.ASC) {
      lines.value.sort((a, b) => {
        if (a[property] === null) return 1;
        if (b[property] === null) return -1;
        return a[property] - b[property];
      });
    } else {
      lines.value.sort((a, b) => {
        if (a[property] === null) return 1;
        if (b[property] === null) return -1;
        return b[property] - a[property];
      });
    }
  }

  if (columnConfig.type === ColsTypeEnum.date) {
    if (order === SortOrder.ASC) {
      lines.value.sort((a, b) => {
        if (a[property] === null) return 1;
        if (b[property] === null) return -1;
        return new Date(a[property]).getTime() - new Date(b[property]).getTime();
      });
    } else {
      lines.value.sort((a, b) => {
        if (a[property] === null) return 1;
        if (b[property] === null) return -1;
        return new Date(b[property]).getTime() - new Date(a[property]).getTime();
      });
    }
  }
}

function onPageSizeChange(newValue: number) {
  pageSize.value = newValue;
  currentPage.value = 1;
  resetPaginationPage.value = true;
  setTimeout(() => (resetPaginationPage.value = false), 100);
  emit('pageSize', newValue);
}

function onPageChange(newValue: number) {
  currentPage.value = newValue;
  emit('page', newValue);
}
</script>

<template>
  <div>
    <div ref="tableContainer" class="table-container" :style="{ 'max-height': `${maxHeightPx}px` ?? auto }">
      <table :class="{ 'fixed-first-col': fixedFirstCol, 'fixed-last-col': fixedLastCol }">
        <thead>
          <tr>
            <TableHeader
              v-for="(columnConfig, index) in config.columnsConfig"
              :key="index"
              :config="columnConfig"
              :width-px="widthsPx[index]"
              :reset-sort="sortedColum?.property !== columnConfig.property"
              :header-translation-prefix="config.headerTranslationPrefix"
              @sort="sort(columnConfig, $event)"
            >
              <template v-if="columnConfig.headerSlot" #[`${columnConfig.property}-header`]>
                <slot :name="`${columnConfig.property}-header`"></slot>
              </template>
            </TableHeader>
          </tr>
        </thead>

        <tbody>
          <tr v-for="(line, index) in lines" :key="index" @click="$emit('rowClick', line)">
            <TableRow
              v-for="(columnConfig, index) in config.columnsConfig"
              :width-px="widthsPx[index]"
              :key="index"
              :config="columnConfig"
              :line="line"
              :index="index"
              :rowClickable="rowClickable"
            >
              <template v-if="columnConfig.type === ColsTypeEnum.slot" #[columnConfig.property!]>
                <slot :name="columnConfig.property" :line="line"></slot>
              </template>
            </TableRow>
          </tr>
        </tbody>
      </table>
    </div>
    <div class="table-bottom" v-if="addPagination">
      <p class="showing">
        {{ t('pagination.showing') }}
        <span>
          {{
            t('pagination.pages', {
              first: currentPage * lines.length - (lines.length - 1),
              last: currentPage * lines.length,
              all: totalCount,
            })
          }}
        </span>
      </p>

      <select
        :value="pageSize"
        class="per-page-selected"
        @change="onPageSizeChange(+($event.target as HTMLSelectElement).value)"
      >
        <option v-for="(page, index) in possiblePageLength" :key="index" :value="page">
          {{ t('pagination.itemsPerPages', { page: page }) }}
        </option>
      </select>

      <TablePaginate :pages-count="pagesCount" :reset-page="resetPaginationPage" @page-changed="onPageChange" />
    </div>
  </div>
</template>

<style scoped lang="scss">
.table-container {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 20px;
  overflow-y: auto;

  table {
    overflow-x: auto;
    border-radius: 5px;
    height: fit-content;
    width: 100%;
    tr {
      background: var(--background-secondary);

      &:nth-child(odd) {
        background: var(--table-even);
      }

      td,
      th {
        padding: 15px 10px;
        text-align: left;
      }

      th {
        background: #555555;
        color: white;
        font-weight: 900;
        font-size: 1.5rem;

        &:first-of-type {
          border-top-left-radius: 5px;
        }

        &:last-of-type {
          border-top-right-radius: 5px;
        }
      }

      &:hover:has(.clickable),
      &:hover td.clickable {
        background: var(--text-light-grey) !important;
        cursor: pointer;
      }
    }

    &.fixed-first-col {
      td:first-child,
      th:first-child {
        position: sticky;
        left: 0;
        filter: drop-shadow(5px 5px 5px rgba(0, 0, 0, 0.1));
        z-index: 1;
      }

      th:first-child {
        background: var(--table-header);
        z-index: 2;
      }

      tr:nth-child(odd) td:first-child {
        background: var(--table-even);
      }

      tr:nth-child(even) td:first-child {
        background: var(--background-secondary);
      }
    }

    &.fixed-last-col {
      td:last-child,
      th:last-child {
        position: sticky;
        right: 0;
        filter: drop-shadow(-5px 5px 5px rgba(0, 0, 0, 0.1));
        z-index: 1;
      }

      th:last-child {
        background: var(--table-header);
        z-index: 1;
      }

      tr:nth-child(odd) td:last-child {
        background: var(--table-even);
      }

      tr:nth-child(even) td:last-child {
        background: var(--background-secondary);
      }
    }
  }
}

.table-bottom {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 20px;
  margin-top: 20px;

  .showing span {
    font-weight: bold;
  }

  .per-page-selected {
    display: flex;
    align-items: center;
    padding: 5px;

    font-size: 1.4rem;
    cursor: pointer;
  }
}
::-webkit-scrollbar-thumb {
  background: var(--table-scrollbar);
}
</style>
