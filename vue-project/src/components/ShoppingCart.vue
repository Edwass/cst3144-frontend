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

    <!-- Only show checkout form if cart has items -->
    <div v-if="groupedItems.length > 0" style="margin-top: 20px;">
      <h3>Checkout</h3>

      <div>
        <label>
          Name:
          <input v-model="name" type="text" />
        </label>
        <div v-if="nameError" style="color: red; font-size: 0.9em;">
          {{ nameError }}
        </div>
      </div>

      <div style="margin-top: 10px;">
        <label>
          Phone:
          <input v-model="phone" type="text" />
        </label>
        <div v-if="phoneError" style="color: red; font-size: 0.9em;">
          {{ phoneError }}
        </div>
      </div>

      <div style="margin-top: 10px;">
        <button
          @click="submitOrder"
          :disabled="!canSubmit"
        >
          Confirm Order
        </button>
      </div>
    </div>
  </section>
</template>

<script setup>
import { computed, ref, watch } from 'vue';

const props = defineProps({
  cartItems: {
    type: Array,
    required: true,
  },
});

const emit = defineEmits(['remove-one', 'checkout']);

/**
 * Group cart items by lessonId and compute quantity.
 * Each grouped item has:
 * { lessonId, topic, location, price, quantity }
 */
const groupedItems = computed(() => {
  const map = new Map();

  for (const item of props.cartItems) {
    const id = item.lessonId ?? item.id; // support both shapes

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

// ----- checkout state & validation -----
const name = ref('');
const phone = ref('');
const nameError = ref('');
const phoneError = ref('');

// simple validators
function validateName(value) {
  if (!value.trim()) return 'Name is required';
  if (value.trim().length < 2) return 'Name must be at least 2 characters';
  return '';
}

function validatePhone(value) {
  if (!value.trim()) return 'Phone is required';
  if (!/^[0-9+\s-]{7,15}$/.test(value.trim())) {
    return 'Phone should contain 7–15 digits (you can include spaces, +, -)';
  }
  return '';
}

// live validation
watch(name, (val) => {
  nameError.value = validateName(val);
});

watch(phone, (val) => {
  phoneError.value = validatePhone(val);
});

const canSubmit = computed(() => {
  return (
    groupedItems.value.length > 0 &&
    !validateName(name.value) &&
    !validatePhone(phone.value)
  );
});

function submitOrder() {
  // final check before emit
  nameError.value = validateName(name.value);
  phoneError.value = validatePhone(phone.value);

  if (nameError.value || phoneError.value) {
    return;
  }

  emit('checkout', {
    name: name.value.trim(),
    phone: phone.value.trim(),
  });

  // optional: reset fields after successful emit
  name.value = '';
  phone.value = '';
}
</script>
