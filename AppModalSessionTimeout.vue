<script setup lang="ts">
import AppModal from '@/components/shared/AppModal.vue';
import { inject, onMounted, ref } from 'vue';
import Keycloak from 'keycloak-js';
import { useI18n } from 'vue-i18n';

const { t } = useI18n();
const keycloak = inject<Keycloak>('keycloak') as Keycloak;

const refreshTokenExpirationDate = ref<Date>(new Date());
const twoMinutesBeforeExpiration = ref<Date>(new Date());

const displayModalResetSession = ref<boolean>(false);
const twoMinutesTimeout = ref<any>();
const remainingSecondsInterval = ref<any>();
const currentNumberOfSecondsBeforeLogout = ref<number>(0);

onMounted(async () => {
  await initTokenTimer();
  document.addEventListener('visibilitychange', handleVisibilityChange);
});

async function initTokenTimer() {
  refreshTokenExpirationDate.value = new Date(JSON.parse(atob(keycloak.refreshToken!.split('.')![1]))?.exp * 1000);
  twoMinutesBeforeExpiration.value = new Date(refreshTokenExpirationDate.value.getTime() - 2 * 60 * 1000);
  
  reset();
  twoMinutesTimeout.value = setTimeout(() => showTimeoutModal(),
    twoMinutesBeforeExpiration.value.getTime() - new Date().getTime());
}

function showTimeoutModal() {
  reset();
  displayModalResetSession.value = true;

  currentNumberOfSecondsBeforeLogout.value = Math.min(
    120,
    Math.floor((refreshTokenExpirationDate.value.getTime() - new Date().getTime()) / 1000),
  );

  remainingSecondsInterval.value = setInterval(() => {
    if (--currentNumberOfSecondsBeforeLogout.value === 0) logout();
  }, 1000);
}

async function refreshToken() {
  await keycloak.updateToken(120);
  localStorage.setItem('kc_token', keycloak.token!);
  localStorage.setItem('kc_refreshToken', keycloak.refreshToken!);

  reset();
  await initTokenTimer();
}

async function logout() {
  reset();

  localStorage.removeItem('kc_token');
  localStorage.removeItem('kc_refreshToken');
  await keycloak?.logout();
}

async function handleVisibilityChange() {
  if (document.visibilityState !== 'visible') return;
  
  if (new Date().getTime() < twoMinutesBeforeExpiration.value.getTime()) {
    await initTokenTimer();
  } else if (new Date().getTime() >= refreshTokenExpirationDate.value.getTime()) {
    await logout();
  } else {
    showTimeoutModal();
  }
}

function reset() {
  displayModalResetSession.value = false;
  clearTimeout(twoMinutesTimeout.value);
  clearInterval(remainingSecondsInterval.value);
}
</script>

<template>
  <AppModal :open="displayModalResetSession" :title="t('session.timeoutTitle')" @close="logout()">
    <template #modal-content>
      {{ t('session.message') }} {{ currentNumberOfSecondsBeforeLogout }} {{ t('session.second') }}.
    </template>
    
    <template #modal-buttons>
      <div class="confirm-modal-footer">
        <button type="button" class="btn btn-primary" @click="refreshToken()">{{ t('session.refresh') }}</button>
      </div>
    </template>
  </AppModal>
</template>

<style scoped>
.confirm-modal-footer {
  display: flex;
}
</style>
