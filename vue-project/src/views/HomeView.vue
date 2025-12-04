<template>
  <main>
    <h1>Lessons Manager</h1>

    <!-- Toggle Buttons -->
    <div>
      <button
  :disabled="!showCart"  
  @click="showCart = false"
>
  Lessons
</button>


      <button
        :disabled="cart.length === 0 || showCart"
        @click="showCart = true"
      >
        Shopping Cart ({{ cart.length }})
      </button>
    </div>

    <!-- Lessons Page -->
    <section v-if="!showCart">
      <LessonForm @lesson-added="fetchLessons" />

      <LessonList
        :lessons="lessons"
        @add-to-cart="handleAddToCart"
      />
    </section>

    <!-- Cart Page -->
    <section v-else>
      <ShoppingCart
        :cart-items="cart"
        @remove-one="handleRemoveFromCart"
      />
    </section>
  </main>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import LessonForm from '../components/LessonForm.vue';
import LessonList from '../components/LessonList.vue';
import ShoppingCart from '../components/ShoppingCart.vue';


// ------------------------------------------
// STATE
// ------------------------------------------
const lessons = ref([]);       // List of all available lessons
const cart = ref([]);          // Items in shopping cart
const showCart = ref(false);   // Toggle lessons <-> cart view


// ------------------------------------------
// FETCH LESSONS FROM BACKEND
// ------------------------------------------
const fetchLessons = async () => {
  const res = await fetch('http://localhost:3000/lessons');
  lessons.value = await res.json();
};

onMounted(fetchLessons);


// ------------------------------------------
// ADD TO CART
// ------------------------------------------
const handleAddToCart = (lessonId) => {
  const lesson = lessons.value.find((l) => l.id === lessonId);
  if (!lesson || lesson.space <= 0) return;

  // Decrease available spaces
  lesson.space--;

  // Add an entry to cart
  cart.value.push({
    cartItemId: Date.now() + Math.random(),
    lessonId: lesson.id,
    topic: lesson.topic,
    location: lesson.location,
    price: lesson.price,
  });
};


// ------------------------------------------
// REMOVE FROM CART (ONE ITEM AT A TIME)
// ------------------------------------------
const handleRemoveFromCart = (lessonId) => {
  // Find index of one matching item
  const index = cart.value.findIndex((c) => c.lessonId === lessonId);
  if (index === -1) return;

  cart.value.splice(index, 1); // remove item

  // Restore space to its lesson
  const lesson = lessons.value.find((l) => l.id === lessonId);
  if (lesson) {
    lesson.space++;
  }

  // If cart becomes empty, return to lessons page
  if (cart.value.length === 0) {
    showCart.value = false;
  }
};
</script>
