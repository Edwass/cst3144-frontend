<template>
  <main>
    <h1>Lessons Manager</h1>

    <LessonForm @lesson-added="handleLessonAdded" />

<LessonList :refresh-key="refreshKey" />

  </main>
</template>

<script setup>
import { ref } from 'vue';
import LessonForm from '../components/LessonForm.vue';
import LessonList from '../components/LessonList.vue';

const refreshKey = ref(0);

const handleLessonAdded = () => {
  refreshKey.value++;
};

</script>
