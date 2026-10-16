<script setup lang="ts">
import { onMounted, useTemplateRef } from "vue";

/** The loading and error screens */
defineProps<{
  title: string;
  message?: string;
  isLoading?: boolean;
}>();
const emit = defineEmits<{ retry: []; changeSettings: [] }>();

const heading = useTemplateRef("heading");
onMounted(() => heading.value?.focus());
</script>

<template>
  <section class="card card--status" :role="isLoading ? 'status' : 'alert'">
    <span v-if="isLoading" class="spinner" aria-hidden="true" />
    <h2 ref="heading" tabindex="-1" class="card__title">{{ title }}</h2>
    <p v-if="message" class="card__text">{{ message }}</p>
    <div v-if="!isLoading" class="actions">
      <button type="button" class="button button--primary" @click="emit('retry')">
        Try again
      </button>
      <button type="button" class="button" @click="emit('changeSettings')">
        Change settings
      </button>
    </div>
  </section>
</template>
