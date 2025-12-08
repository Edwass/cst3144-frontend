<template>
  <form class="form" @submit.prevent="submitForm">
    <div class="form-row form-row--inline">
      <label class="field-label">
        Subject / Topic
        <input v-model="topic" class="field-input" required />
      </label>

      <label class="field-label">
        Location
        <input v-model="location" class="field-input" required />
      </label>
    </div>

    <div class="form-row form-row--inline">
      <label class="field-label">
        Price (£)
        <input
          v-model.number="price"
          type="number"
          min="0"
          class="field-input"
          required
        />
      </label>

      <label class="field-label">
        Spaces
        <input
          v-model.number="space"
          type="number"
          min="0"
          class="field-input"
          required
        />
      </label>
    </div>

    <div class="form-row form-row--right">
      <button type="submit" class="btn btn--primary">
        Add Lesson
      </button>
    </div>
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
      space: space.value,
    }),
  });

  if (!response.ok) {
    console.error('Failed to create lesson');
    return;
  }

  await response.json();
  emit('lesson-added');

  topic.value = '';
  location.value = '';
  price.value = 0;
  space.value = 0;
};
</script>
