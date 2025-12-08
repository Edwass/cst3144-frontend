<template>
  <div v-if="visible" class="toast" :class="type">
    {{ message }}
  </div>
</template>

<script setup>
import { ref } from 'vue';

const visible = ref(false);
const message = ref('');
const type = ref('success');

function show(msg, variant = 'success') {
  message.value = msg;
  type.value = variant;
  visible.value = true;

  setTimeout(() => {
    visible.value = false;
  }, 2500);
}

defineExpose({ show });
</script>

<style scoped>
.toast {
  position: fixed;
  top: 25px;
  right: 25px;
  padding: 14px 20px;
  border-radius: 8px;
  color: white;
  font-size: 0.95rem;
  font-weight: 500;
  box-shadow: 0 4px 20px rgba(0,0,0,0.15);
  animation: slideDown 0.25s ease-out;
  z-index: 9999;
}
.toast.success { background: #1fbf75; }
.toast.error { background: #e74c3c; }

@keyframes slideDown {
  from { transform: translateY(-20px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}
</style>
