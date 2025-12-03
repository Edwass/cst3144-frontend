<template>
  <section>
    <h2>Lessons</h2>
    <ul>
      <li v-for="lesson in lessons" :key="lesson.id">
        <strong>{{ lesson.topic }}</strong>
        <span> – {{ lesson.location }}</span>
        <div>
          Price: £{{ lesson.price }}
          &nbsp;|&nbsp;
          Spaces left: {{ lesson.space }}
        </div>
      </li>
    </ul>
  </section>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';

const props = defineProps({
  refreshKey: {
    type: Number,
    default: 0
  }
});

const lessons = ref([]);

const fetchLessons = async () => {
  const res = await fetch('http://localhost:3000/lessons');
  lessons.value = await res.json();
};

onMounted(fetchLessons);
watch(
  () => props.refreshKey,
  () => {
    fetchLessons();
  }
);
</script>
