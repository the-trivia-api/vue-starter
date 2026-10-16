<script setup lang="ts">
import { reactive } from "vue";
import type { QuizSettings } from "../api";
import { CATEGORIES, CATEGORY_LABELS } from "../categories";

const props = defineProps<{ defaultSettings: QuizSettings }>();
const emit = defineEmits<{ start: [settings: QuizSettings] }>();

// A copy, so changing a select doesn't change the settings until the quiz starts
const form = reactive<QuizSettings>({ ...props.defaultSettings });

const AMOUNTS = [5, 10, 15, 20];
</script>

<template>
  <form class="card" @submit.prevent="emit('start', { ...form })">
    <h2 class="card__title">Set up your quiz</h2>

    <div class="fields">
      <label class="field">
        <span class="field__label">Category</span>
        <select v-model="form.category" class="select">
          <option value="any">Any category</option>
          <option v-for="category in CATEGORIES" :key="category" :value="category">
            {{ CATEGORY_LABELS[category] }}
          </option>
        </select>
      </label>

      <label class="field">
        <span class="field__label">Difficulty</span>
        <select v-model="form.difficulty" class="select">
          <option value="any">Any difficulty</option>
          <option value="easy">Easy</option>
          <option value="medium">Medium</option>
          <option value="hard">Hard</option>
        </select>
      </label>

      <label class="field">
        <span class="field__label">Questions</span>
        <select v-model.number="form.amount" class="select">
          <option v-for="amount in AMOUNTS" :key="amount" :value="amount">{{ amount }}</option>
        </select>
      </label>
    </div>

    <button type="submit" class="button button--primary">Start quiz</button>
  </form>
</template>
