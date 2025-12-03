<template>
  <section>
    <h2>Add New Lesson</h2>
    <form @submit.prevent="handleSubmit">
      <div>
        <label>
          Title:
          <input v-model="title" required />
        </label>
      </div>

      <div>
        <label>
          Description:
          <textarea v-model="description" required></textarea>
        </label>
      </div>

      <div>
        <label>
          Date:
          <input type="date" v-model="date" required />
        </label>
      </div>

      <button type="submit" :disabled="submitting">
        {{ submitting ? 'Saving...' : 'Add Lesson' }}
      </button>

      <p v-if="error" style="color: red;">{{ error }}</p>
      <p v-if="success" style="color: green;">Lesson added successfully.</p>
    </form>
  </section>
</template>

<script setup>
import { ref } from 'vue';

const emit = defineEmits(['lesson-added']);

const title = ref('');
const description = ref('');
const date = ref('');
const submitting = ref(false);
const error = ref(null);
const success = ref(false);

const handleSubmit = async () => {
  submitting.value = true;
  error.value = null;
  success.value = false;

  try {
    const response = await fetch('http://localhost:3000/api/lessons', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        title: title.value,
        description: description.value,
        date: date.value
      })
    });

    if (!response.ok) {
      throw new Error('Failed to create lesson');
    }

    await response.json();

    // clear fields
    title.value = '';
    description.value = '';
    date.value = '';

    success.value = true;

    // notify parent
    emit('lesson-added');
  } catch (err) {
    error.value = err.message || 'Something went wrong while creating lesson';
  } finally {
    submitting.value = false;
  }
};
</script>
