<template>
  <div>
    <h2>Lessons</h2>

    <ul>
      <li v-for="lesson in lessons" :key="lesson.id">
        <strong>{{ lesson.title }}</strong> – {{ lesson.date }} <br />
        {{ lesson.description }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';

const props = defineProps({
  refreshKey: Number
});

const lessons = ref([]);

const loadLessons = async () => {
  const res = await fetch('http://localhost:3000/lessons');
  lessons.value = await res.json();
};

onMounted(loadLessons);

watch(
  () => props.refreshKey,
  () => {
    loadLessons();
  }
);
</script>
