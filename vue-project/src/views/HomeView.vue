<template>
  <main class="app">
    <header class="app-header">
      <h1 class="app-title">Lessons Manager</h1>

      <!-- Toggle Buttons -->
      <nav class="tabs">
        <button
          class="tab"
          :class="{ 'tab--active': !showCart }"
          :disabled="!showCart"
          @click="showCart = false"
        >
          Lessons
        </button>

        <button
          class="tab"
          :class="{ 'tab--active': showCart }"
          :disabled="cart.length === 0 || showCart"
          @click="showCart = true"
        >
          Shopping Cart ({{ cart.length }})
        </button>
      </nav>
    </header>

    <!-- Lessons Page -->
    <section v-if="!showCart" class="layout">
      <section class="card card--form">
        <h2 class="card-title">Add New Lesson</h2>
        <LessonForm @lesson-added="fetchLessons" />
      </section>

      <section class="card card--tools">
        <!-- Search box (search as you type) -->
        <div class="form-row">
          <label class="field-label">
            Search
            <input
              v-model="searchTerm"
              type="text"
              class="field-input"
              placeholder="Type to search subject, location, price, spaces..."
            />
          </label>
        </div>

        <!-- Sort controls -->
        <div class="form-row form-row--inline">
          <label class="field-label">
            Sort by
            <select v-model="sortBy" class="field-input">
              <option value="topic">Subject</option>
              <option value="location">Location</option>
              <option value="price">Price</option>
              <option value="space">Spaces</option>
            </select>
          </label>

          <label class="field-label">
            Order
            <select v-model="sortOrder" class="field-input">
              <option value="asc">Ascending</option>
              <option value="desc">Descending</option>
            </select>
          </label>
        </div>
      </section>

      <section class="card card--lessons">
        <LessonList
          :lessons="sortedLessons"
          @add-to-cart="handleAddToCart"
        />
      </section>
    </section>

    <!-- Cart Page -->
    <section v-else class="layout">
      <section class="card card--cart">
        <ShoppingCart
          :cart-items="cart"
          @remove-one="handleRemoveFromCart"
          @checkout="handleCheckout"
        />
      </section>
    </section>
  </main>
</template>

<script setup>
import { ref, onMounted, computed, watch, inject } from 'vue';
import LessonForm from '../components/LessonForm.vue';
import LessonList from '../components/LessonList.vue';
import ShoppingCart from '../components/ShoppingCart.vue';

// ------------------------------------------
// STATE
// ------------------------------------------
const lessons = ref([]);       // List of all available lessons (from API/search)
const cart = ref([]);          // Items in shopping cart
const showCart = ref(false);   // Toggle lessons <-> cart view

const sortBy = ref('topic');
const sortOrder = ref('asc');
const searchTerm = ref('');

// Toast instance provided from App.vue
const toast = inject('toast');

const API_BASE = 'http://localhost:3000';

// Helper: normalise lessons so they always have an "id" string
const normaliseLessons = (rawArray) =>
  rawArray.map((doc) => ({
    id: doc.id ?? doc._id?.toString(),   // use "id" if present, otherwise Mongo _id
    topic: doc.topic,
    location: doc.location,
    price: doc.price,
    space: doc.space,
  }));

// ------------------------------------------
// FETCH ALL LESSONS (GET /lessons)
// ------------------------------------------
const fetchLessons = async () => {
  const res = await fetch(`${API_BASE}/lessons`);
  const data = await res.json();
  lessons.value = normaliseLessons(data);
};

// ------------------------------------------
// SEARCH LESSONS (GET /search?q=...)
// ------------------------------------------
const fetchSearchResults = async () => {
  const q = searchTerm.value.trim();

  const url = q
    ? `${API_BASE}/search?q=${encodeURIComponent(q)}`
    : `${API_BASE}/lessons`;

  const res = await fetch(url);
  const data = await res.json();
  lessons.value = normaliseLessons(data);
};

// Initial load
onMounted(fetchLessons);

// "Search as you type": whenever searchTerm changes, hit /search
watch(searchTerm, () => {
  fetchSearchResults();
});

// ------------------------------------------
// SORTED LESSONS (by subject/location/price/space)
// ------------------------------------------
const sortedLessons = computed(() => {
  const copy = [...lessons.value];

  copy.sort((a, b) => {
    let valA = a[sortBy.value];
    let valB = b[sortBy.value];

    // String compare for topic / location
    if (typeof valA === 'string') {
      valA = valA.toLowerCase();
      valB = valB.toLowerCase();
      if (valA < valB) return sortOrder.value === 'asc' ? -1 : 1;
      if (valA > valB) return sortOrder.value === 'asc' ? 1 : -1;
      return 0;
    }

    // Numeric compare for price / space
    if (valA < valB) return sortOrder.value === 'asc' ? -1 : 1;
    if (valA > valB) return sortOrder.value === 'asc' ? 1 : -1;
    return 0;
  });

  return copy;
});

// ------------------------------------------
// ADD TO CART
// ------------------------------------------
const handleAddToCart = (lessonId) => {
  const lesson = lessons.value.find((l) => l.id === lessonId);
  if (!lesson || lesson.space <= 0) return;

  // Decrease available spaces locally
  lesson.space--;

  // Add an entry to cart
  cart.value.push({
    lessonId: lesson.id,
    topic: lesson.topic,
    location: lesson.location,
    price: lesson.price,
  });
};

// ------------------------------------------
// REMOVE ONE FROM CART
// ------------------------------------------
const handleRemoveFromCart = (lessonId) => {
  const index = cart.value.findIndex((c) => c.lessonId === lessonId);
  if (index === -1) return;

  cart.value.splice(index, 1);

  const lesson = lessons.value.find((l) => l.id === lessonId);
  if (lesson) {
    lesson.space++;
  }

  if (cart.value.length === 0) {
    showCart.value = false;
  }
};

// ------------------------------------------
// CHECKOUT: POST /orders (backend updates spaces)
// ------------------------------------------
const handleCheckout = async ({ name, phone }) => {
  try {
    // Group cart items by lessonId
    const map = new Map();
    for (const item of cart.value) {
      const id = item.lessonId;
      map.set(id, (map.get(id) || 0) + 1);
    }

    const items = Array.from(map.entries()).map(([lessonId, quantity]) => ({
      lessonId,
      quantity,
    }));

    const totalSpaces = items.reduce((sum, it) => sum + it.quantity, 0);

    // Build order object (same shape as before)
    const orderPayload = {
      name,
      phone,
      lessonIDs: items.map((it) => it.lessonId),
      spaces: totalSpaces,
      items,
      createdAt: new Date().toISOString(),
    };

    // 1) Create order – backend will also update lesson spaces
    const orderRes = await fetch(`${API_BASE}/orders`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(orderPayload),
    });

    if (!orderRes.ok) {
      // Try to read error message from backend
      let message = 'Failed to submit order. Please try again.';
      try {
        const body = await orderRes.json();
        if (body && body.error) {
          message = body.error;
        }
      } catch {
        // ignore JSON errors
      }

      // Toast error instead of alert
      toast?.value?.show(message, 'error');
      return;
    }

    // 2) Clear cart and go back to lessons
    cart.value = [];
    showCart.value = false;

    // 3) Refresh lessons from DB so spaces are up to date
    await fetchLessons();

    // Toast success message
    toast?.value?.show('Order submitted successfully!', 'success');
  } catch (err) {
    console.error('Checkout error:', err);
    // Toast generic error
    toast?.value?.show('Something went wrong during checkout.', 'error');
  }
};
</script>
