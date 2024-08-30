<script setup lang="ts">
import { ref, watch } from 'vue';
import { useI18n } from 'vue-i18n';

const emit = defineEmits(['change']);

const props = withDefaults(
  defineProps<{
    tabs: string[];
    tab: string;
    translationPrefix: string;
  }>(),
  {}
);

const { t } = useI18n();
const currentTab = ref<string>(props.tabs[0]);

watch(
  () => props.tab,
  () => {
    currentTab.value = props.tab;
  },
  { immediate: true }
);

function changeTab(tab: string) {
  if (currentTab.value === tab) return;
  currentTab.value = tab;
  emit('change', tab);
}
</script>

<template>
  <div class="tabs-container">
    <div class="tabs-items">
      <h3
        @click="changeTab(tab)"
        v-for="(tab, index) in props.tabs"
        :key="index"
        :class="{ active: currentTab === tab }"
      >
        {{ t(`${translationPrefix}.${tab}`) }}
      </h3>
    </div>

    <div :key="currentTab" class="tab-content">
      <slot :name="currentTab"></slot>
    </div>
  </div>
</template>

<style scoped lang="scss">
.tabs-container {
  display: flex;
  flex-direction: column;
  gap: 13px;
  width: 100%;

  .tabs-items {
    display: flex;
    align-items: center;
    gap: 20px;

    h3 {
      cursor: pointer;

      &.active {
        text-decoration-thickness: 2px;
        text-decoration-line: underline;
        text-underline-offset: 7px;
        text-decoration-color: var(--primary-color);
      }
    }
  }
}
</style>
