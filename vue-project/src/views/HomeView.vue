<template>
  <main>
    <h1>Lessons Manager</h1>

    <!-- Cart toggle -->
    <div style="margin-bottom: 1rem;">
      <button
        :disabled="cart.length === 0"
        @click="toggleCart"
      >
        {{ showCart ? 'Back to Lessons' : `View Cart (${cart.length})` }}
      </button>
    </div>

    <!-- Lessons + form page -->
    <section v-if="!showCart">
      <LessonForm @lesson-added="handleLessonAdded" />

      <LessonList
        :lessons="lessons"
        @add-to-cart="handleAddToCart"
      />
    </section>

    <!-- Cart page -->
    <section v-else>
      <h2>Shopping Cart</h2>

      <p v-if="cart.length === 0">Your cart is empty.</p>

      <ul v-else>
        <li v-for="(item, index) in cart" :key="item.cartItemId">
          <strong>{{ item.topic }}</strong>
          – {{ item.location }} – £{{ item.price }}
          <button @click="removeFromCart(index, item.lessonId)">
            Remove
          </button>
        </li>
      </ul>
    </section>
  </main>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import LessonForm from '../components/LessonForm.vue';
import LessonList from '../components/LessonList.vue';

// ---- state ----
const lessons = ref([]);      // all lessons shown in list
const cart = ref([]);         // items added to cart
const showCart = ref(false);  // toggle between lessons & cart pages

// ---- load lessons from backend ----
const fetchLessons = async () => {
  const res = await fetch('http://localhost:3000/lessons');
  lessons.value = await res.json();
};

onMounted(fetchLessons);

// called when LessonForm emits "lesson-added"
const handleLessonAdded = async () => {
  await fetchLessons();
};

// called when LessonList emits "add-to-cart"
const handleAddToCart = (lessonId) => {
  const lesson = lessons.value.find((l) => l.id === lessonId);
  if (!lesson || lesson.space <= 0) return;

  // decrease available spaces
  lesson.space--;

  // add one instance to cart
  cart.value.push({
    cartItemId: Date.now() + Math.random(), // unique key for v-for
    lessonId: lesson.id,
    topic: lesson.topic,
    location: lesson.location,
    price: lesson.price,
  });
};

// remove a single instance from cart and give space back
const removeFromCart = (index, lessonId) => {
  cart.value.splice(index, 1);

  const lesson = lessons.value.find((l) => l.id === lessonId);
  if (lesson) {
    lesson.space++;
  }

  if (cart.value.length === 0) {
    showCart.value = false;
  }
};

const toggleCart = () => {
  if (cart.value.length === 0) return;
  showCart.value = !showCart.value;
};
</script>
