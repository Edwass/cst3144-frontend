<template>
  <section class="lessons-section">
    <h2 class="section-title">Lessons</h2>

    <ul class="lessons-grid">
      <li
        v-for="lesson in lessons"
        :key="lesson.id"
        class="lesson-card"
      >
        <header class="lesson-card__header">
          <h3 class="lesson-card__title">
            {{ lesson.topic }}
          </h3>
          <p class="lesson-card__location">
            {{ lesson.location }}
          </p>
        </header>

        <p class="lesson-card__meta">
          <span>Price: <strong>£{{ lesson.price }}</strong></span>
          <span>Spaces left: <strong>{{ lesson.space }}</strong></span>
        </p>

        <button
          class="btn btn--secondary btn--full"
          :disabled="lesson.space === 0"
          @click="$emit('add-to-cart', lesson.id)"
        >
          <span v-if="lesson.space === 0">Fully booked</span>
          <span v-else>Add to Cart</span>
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

