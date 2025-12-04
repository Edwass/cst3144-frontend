<template>
  <section>
    <h2>Lessons</h2>
    <ul>
      <li v-for="lesson in lessons" :key="lesson.id">
        <strong>{{ lesson.topic }}</strong> – {{ lesson.location }}
        <br />
        Price: £{{ lesson.price }} | Spaces left: {{ lesson.space }}
        <br />
        <button
          :disabled="lesson.space === 0"
          @click="$emit('add-to-cart', lesson.id)"
        >
          Add to Cart
        </button>
      </li>
    </ul>
  </section>
</template>

<script setup>
defineProps({
  lessons: {
    type: Array,
    required: true,
  },
});
</script>
