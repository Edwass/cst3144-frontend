<template>
  <div id="app">
    <router-view />

    <!-- Global Toast -->
    <ToastMessage ref="toast" />
  </div>
</template>

<script setup>
import { ref, provide } from 'vue';
import ToastMessage from './components/ToastMessage.vue';

const toast = ref(null);
provide('toast', toast);
</script>
