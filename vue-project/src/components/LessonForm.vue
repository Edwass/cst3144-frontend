<template>
  <form @submit.prevent="submitForm">
    <label>Subject / Topic:</label>
    <input v-model="topic" required />

    <label>Location:</label>
    <input v-model="location" required />

    <label>Price (£):</label>
    <input type="number" v-model.number="price" min="0" required />

    <label>Spaces:</label>
    <input type="number" v-model.number="space" min="0" required />

    <button type="submit">Add Lesson</button>
  </form>
</template>

<script setup>
import { ref } from 'vue';

const emit = defineEmits(['lesson-added']);

const topic = ref('');
const location = ref('');
const price = ref(0);
const space = ref(0);

const submitForm = async () => {
  const response = await fetch('http://localhost:3000/lessons', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      topic: topic.value,
      location: location.value,
      price: price.value,
      space: space.value
    })
  });

  if (!response.ok) {
    console.error('Failed to create lesson');
    return;
  }

  await response.json();
  emit('lesson-added'); // notify parent to refresh

  // Reset form
  topic.value = '';
  location.value = '';
  price.value = 0;
  space.value = 0;
};
</script>
