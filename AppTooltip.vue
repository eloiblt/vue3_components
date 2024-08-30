<script setup lang="ts">
import { ref, watch } from 'vue';

const props = withDefaults(
  defineProps<{
    // the text inside the tooltip
    tooltipText?: string;

    // show/hide tooltip, default true
    showTooltip?: boolean;

    // in manual mode, the tooltip will be shown depending on the open props
    // default false
    manual?: boolean;
    open?: boolean;

    // position props
    left?: number;
    right?: number;
    top?: number;
    bottom?: number;
  }>(),
  {
    showTooltip: true,
    manual: false,
  }
);

const hovered = ref<boolean>(false);

watch(
  () => props.open,
  () => {
    hovered.value = props.open;
  }
);

function mouseEnter() {
  if (props.manual) return;
  hovered.value = true;
}

function mouseLeave() {
  if (props.manual) return;
  hovered.value = false;
}
</script>

<template>
  <div class="tooltip-container" @mouseenter="mouseEnter" @mouseleave="mouseLeave">
    <slot name="hoverable-content" />

    <p
      class="tooltip-text"
      v-if="showTooltip && hovered"
      :style="{ left: `${left}px`, right: `${right}px`, top: `${top}px`, bottom: `${bottom}px` }"
    >
      {{ tooltipText }}
    </p>
  </div>
</template>

<style scoped lang="scss">
.tooltip-container {
  position: relative;
  width: 100%;

  .tooltip-text {
    position: absolute;
    overflow: hidden;
    word-break: break-word;
    background-color: var(--background-secondary);
    box-shadow: 0 0 20px -5px var(--card-shadow);
    color: var(--text-darker-grey);
    font-family: BellSlim, serif;
    border-radius: 6px;
    z-index: 3;
    padding: 5px 10px;
  }
}
</style>
