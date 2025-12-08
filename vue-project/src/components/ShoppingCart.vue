<template>
  <section>
    <h2 class="section-title">Shopping Cart</h2>

    <p v-if="groupedItems.length === 0" class="muted">
      Your cart is empty.
    </p>

    <ul v-else class="cart-list">
      <li
        v-for="item in groupedItems"
        :key="item.lessonId"
        class="cart-item"
      >
        <div class="cart-item__info">
          <h3 class="cart-item__title">
            {{ item.topic }}
          </h3>
          <p class="cart-item__subtitle">
            {{ item.location }}
          </p>

          <p class="cart-item__meta">
            Price: <strong>£{{ item.price }}</strong> |
            Quantity: <strong>{{ item.quantity }}</strong>
          </p>
        </div>

        <button
          class="btn btn--ghost"
          @click="$emit('remove-one', item.lessonId)"
        >
          Remove one
        </button>
      </li>
    </ul>

    <!-- Checkout form -->
    <div v-if="groupedItems.length > 0" class="checkout">
      <h3 class="section-subtitle">Checkout</h3>

      <div class="form-row">
        <label class="field-label">
          Name
          <input v-model="name" type="text" class="field-input" />
        </label>
        <p v-if="nameError" class="error-text">{{ nameError }}</p>
      </div>

      <div class="form-row">
        <label class="field-label">
          Phone
          <input v-model="phone" type="text" class="field-input" />
        </label>
        <p v-if="phoneError" class="error-text">{{ phoneError }}</p>
      </div>

      <div class="form-row form-row--right">
        <button
          class="btn btn--primary"
          :disabled="!canSubmit"
          @click="submitOrder"
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

// group cart items by lessonId
const groupedItems = computed(() => {
  const map = new Map();

  for (const item of props.cartItems) {
    const key = item.lessonId ?? item.id;
    const existing = map.get(key);

    if (!existing) {
      map.set(key, {
        lessonId: key,
        topic: item.topic,
        location: item.location,
        price: item.price,
        quantity: 1,
      });
    } else {
      existing.quantity += 1;
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
  nameError.value = validateName(name.value);
  phoneError.value = validatePhone(phone.value);

  if (nameError.value || phoneError.value) return;

  emit('checkout', {
    name: name.value.trim(),
    phone: phone.value.trim(),
  });

  name.value = '';
  phone.value = '';
}
</script>
