<template>
  <section>
    <h2>Shopping Cart</h2>

    <p v-if="groupedItems.length === 0">
      Your cart is empty.
    </p>

    <ul v-else>
      <li v-for="item in groupedItems" :key="item.lessonId">
        <strong>{{ item.topic }}</strong> – {{ item.location }}
        <br>
        Price: £{{ item.price }} |
        Quantity: {{ item.quantity }}
        <br>
        <button @click="$emit('remove-one', item.lessonId)">
          Remove one
        </button>
      </li>
    </ul>

    <!-- Checkout form (only if we have items) -->
    <section v-if="groupedItems.length > 0" style="margin-top: 1rem;">
      <h3>Checkout</h3>

      <form @submit.prevent="onCheckout">
        <div>
          <label>Name:</label>
          <input v-model="name" required />
        </div>

        <div>
          <label>Phone:</label>
          <input v-model="phone" required />
        </div>

        <p v-if="name && !isNameValid" style="color: red;">
          Name must contain letters and spaces only.
        </p>
        <p v-if="phone && !isPhoneValid" style="color: red;">
          Phone must contain digits only.
        </p>

        <button type="submit" :disabled="!isFormValid">
          Checkout
        </button>
      </form>
    </section>
  </section>
</template>

<script setup>
import { computed, ref } from 'vue';

const props = defineProps({
  cartItems: {
    type: Array,
    required: true,
  },
});

const emit = defineEmits(['remove-one', 'checkout']);

// --------------------------------------------------
// Group cart items by lessonId
// --------------------------------------------------
const groupedItems = computed(() => {
  const map = new Map();

  for (const item of props.cartItems) {
    const id = item.lessonId; // we store lessonId in cart
    if (!map.has(id)) {
      map.set(id, {
        lessonId: id,
        topic: item.topic,
        location: item.location,
        price: item.price,
        quantity: 1,
      });
    } else {
      map.get(id).quantity += 1;
    }
  }

  return Array.from(map.values());
});

// --------------------------------------------------
// Checkout form state + validation
// --------------------------------------------------
const name = ref('');
const phone = ref('');

const isNameValid = computed(() => /^[A-Za-z\s]+$/.test(name.value));
const isPhoneValid = computed(() => /^\d+$/.test(phone.value));

const isFormValid = computed(
  () => groupedItems.value.length > 0 && isNameValid.value && isPhoneValid.value
);

// --------------------------------------------------
// Emit checkout to parent
// --------------------------------------------------
const onCheckout = () => {
  if (!isFormValid.value) return;

  emit('checkout', {
    name: name.value.trim(),
    phone: phone.value.trim(),
  });
};
</script>
