<template>
  <form @submit.prevent="submitForm">
    <label>Title:</label>
    <input v-model="title" required />

    <label>Description:</label>
    <input v-model="description" required />

    <label>Date:</label>
    <input type="date" v-model="date" required />

    <button type="submit">Add Lesson</button>
  </form>
</template>

<script setup>
import { ref } from 'vue';

const title = ref('');
const description = ref('');
const date = ref('');

const emit = defineEmits(['lesson-added']);

const submitForm = async () => {
  const response = await fetch('http://localhost:3000/lessons', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      title: title.value,
      description: description.value,
      date: date.value
    })
  });

  await response.json();

  // Tell parent to refresh the list
  emit('lesson-added');

  // Reset form
  title.value = '';
  description.value = '';
  date.value = '';
};
</script>
