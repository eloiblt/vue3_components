<script setup lang="ts">
import { ref } from 'vue';
import { useI18n } from 'vue-i18n';
import AppToggle from '@/components/shared/AppToggle.vue';

const { t } = useI18n();
const emit = defineEmits(['checked']);

const props = withDefaults(
  defineProps<{
    menu: string[];
    translationPrefix: string;
    menuChecked?: boolean[];
  }>(),
  {}
);

const openedMenu = ref<Set<string>>(new Set([...props.menu]));
</script>

<template>
  <div class="accordion-container">
    <div class="item" v-for="(item, index) in props.menu" :key="index">
      <div class="title-container">
        <div class="title" @click="openedMenu.has(item) ? openedMenu.delete(item) : openedMenu.add(item)">
          <Transition name="fade">
            <div class="title-icon" v-if="!openedMenu.has(item)">
              <img src="../../assets/plus.svg" alt="plus" />
            </div>
          </Transition>
          <Transition name="fade">
            <div class="title-icon" v-if="openedMenu.has(item)">
              <img src="../../assets/minus.svg" alt="minus" />
            </div>
          </Transition>
          <p>{{ t(`${translationPrefix}.${item}`) }}</p>
        </div>

        <AppToggle
          v-if="menuChecked"
          :force-state="menuChecked[index]"
          @change="emit('checked', { item, checked: $event })"
        />
      </div>

      <div class="content" :class="{ open: openedMenu.has(item) }">
        <slot :name="item"></slot>
      </div>
    </div>
  </div>
</template>

<style scoped lang="scss">
.accordion-container {
  display: flex;
  flex-direction: column;

  .item {
    display: flex;
    flex-direction: column;
    border-top: 1px solid var(--light-separator);
    padding: 18px 30px;

    .title-container {
      cursor: pointer;
      position: relative;
      height: 20px;
      display: flex;
      align-items: center;
      width: 100%;
      user-select: none;

      .title {
        display: flex;
        align-items: center;
        flex: 1;

        .title-icon {
          position: absolute;
          inset: 0;
          color: var(--primary-color);
          width: fit-content;

          img {
            width: 20px;
          }
        }

        > p {
          margin-left: 30px;
          font-weight: 700;
          font-size: 1.8rem;
        }
      }

      :deep(.switch) {
        //margin-left: auto;
      }
    }

    .content {
      max-height: 0;
      overflow: hidden;
      transition: max-height 0.3s cubic-bezier(0, 1, 0, 1) -0.1s;
      margin-left: 30px;

      &:before {
        content: '';
        display: block;
        height: 20px;
      }

      &.open {
        max-height: 9999px;
        transition-timing-function: cubic-bezier(0.5, 0, 1, 0);
        transition-delay: 0s;
      }
    }

    &:last-of-type {
      border-bottom: 1px solid var(--light-separator);
    }
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s linear;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
