<script setup lang="ts">
import { type ColumnConfig } from '@/models/table-data.model';
import { useI18n } from 'vue-i18n';
import AppTooltip from '@/components/shared/AppTooltip.vue';
import { ColsTypeEnum } from '@/enums/cols-type.enum.ts';
import { formatDate } from '@/helpers/format-date.helper.ts';

const props = withDefaults(
  defineProps<{
    config: ColumnConfig;
    line: any;
    index: number;
    widthPx: number;
    rowClickable?: boolean;
  }>(),
  {}
);

const { t } = useI18n();

function formatDatailUrl() {
  const key = props.config.link?.substring(props.config.link.indexOf('{') + 1, props.config.link.lastIndexOf('}'))!;
  return props.config.link?.replace(`{${key}}`, props.line[key])!;
}
</script>

<template>
  <td
    :style="[widthPx ? { 'max-width': `${widthPx}px`, width: `${widthPx}px` } : null]"
    :class="rowClickable ? 'clickable' : ''"
  >
    <div class="td-container">
      <AppTooltip
        v-if="[ColsTypeEnum.text, ColsTypeEnum.decimal].includes(config?.type!)"
        :tooltip-text="line[config.property!]"
        :show-tooltip="config?.showTooltip ?? false"
        :left="-10"
        :bottom="25"
      >
        <template #hoverable-content>
          <p>
            <template v-if="config.rowTranslationPrefix">{{
              t(`${config.rowTranslationPrefix}.${line[config.property!]}`)
            }}</template>
            <template v-else>{{ line[config.property!] }}</template>
          </p>
        </template>
      </AppTooltip>

      <p v-else-if="config.type === ColsTypeEnum.date">{{ formatDate(line[config.property!]) }}</p>

      <router-link v-else-if="config.type === ColsTypeEnum.link" :to="formatDatailUrl()">
        {{ t(`tablesLabels.${config.linkLabel}`) }}
      </router-link>

      <i v-else-if="config.type === ColsTypeEnum.icon" class="fa-solid" :class="config.iconClass"></i>

      <slot v-else-if="config.type === ColsTypeEnum.slot" :name="config.property" :line="line"></slot>
    </div>
  </td>
</template>

<style scoped lang="scss">
@import '@/styles/mixins';

td {
  vertical-align: middle;

  .td-container {
    position: relative;

    p {
      color: var(--text-darker-grey);
      @include ellispsis;
    }

    a {
      color: var(--text-primary);
      text-decoration: underline;
      font-weight: bold;
    }
  }
}
</style>
