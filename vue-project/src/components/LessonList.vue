<template>
  <section>
    <h2>Lessons</h2>

    <p v-if="loading">Loading lessons...</p>
    <p v-else-if="error" style="color: red;">{{ error }}</p>

    <ul v-else>
      <li v-for="lesson in lessons" :key="lesson.id">
        <strong>{{ lesson.title }}</strong> – {{ lesson.date }}<br />
        <small>{{ lesson.description }}</small>
      </li>
    </ul>
  </section>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const lessons = ref([]);
const loading = ref(true);
const error = ref(null);

const fetchLessons = async () => {
  loading.value = true;
  error.value = null;

  try {
    const response = await fetch('http://localhost:3000/api/lessons');
    if (!response.ok) {
      throw new Error('Failed to load lessons');
    }
    lessons.value = await response.json();
  } catch (err) {
    error.value = err.message || 'Something went wrong while loading lessons';
  } finally {
    loading.value = false;
  }
};

onMounted(fetchLessons);
</script>
