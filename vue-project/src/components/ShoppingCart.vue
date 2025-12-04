<template>
  <section>
    <h2>Shopping Cart</h2>

    <p v-if="groupedItems.length === 0">
      Your cart is empty.
    </p>

    <ul v-else>
      <li
        v-for="item in groupedItems"
        :key="item.lessonId"
      >
        <strong>{{ item.lesson.topic }}</strong> – {{ item.lesson.location }}
        <br>
        Price: £{{ item.lesson.price }} |
        Quantity: {{ item.quantity }}
        <br>
        <button @click="$emit('remove-one', item.lessonId)">
          Remove one
        </button>
      </li>
    </ul>
  </section>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  cartItems: {
    type: Array,
    required: true,
  },
});

/*
  cartItems elements look like:
  {
    cartItemId: ...,
    lessonId: number,
    topic: string,
    location: string,
    price: number
  }

  We group by lessonId and keep a count.
*/
const groupedItems = computed(() => {
  const map = new Map();

  for (const item of props.cartItems) {
    const id = item.lessonId;

    if (!map.has(id)) {
      map.set(id, {
        lessonId: id,
        lesson: item,   // holds topic, location, price
        quantity: 1,
      });
    } else {
      map.get(id).quantity += 1;
    }
  }

  return Array.from(map.values());
});
</script>
