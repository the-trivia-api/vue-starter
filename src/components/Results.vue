<script setup lang="ts">
import { computed, onMounted, useTemplateRef } from "vue";
import { isCorrect, type Round } from "../types";

const props = defineProps<{ rounds: Round[] }>();
const emit = defineEmits<{ playAgain: []; changeSettings: [] }>();

const score = computed(() => props.rounds.filter(isCorrect).length);

const verdict = computed(() => {
  const ratio = score.value / props.rounds.length;
  if (ratio === 1) return "A perfect score.";
  if (ratio >= 0.7) return "Great work.";
  if (ratio >= 0.4) return "Not bad at all.";
  return "Better luck next time.";
});

const heading = useTemplateRef("heading");
onMounted(() => heading.value?.focus());
</script>

<template>
  <section class="card" aria-labelledby="results-title">
    <h2 id="results-title" ref="heading" tabindex="-1" class="card__title">
      You scored {{ score }} out of {{ rounds.length }}
    </h2>
    <p class="card__text">{{ verdict }}</p>

    <div class="actions">
      <button type="button" class="button button--primary" @click="emit('playAgain')">
        Play again
      </button>
      <button type="button" class="button" @click="emit('changeSettings')">
        Change settings
      </button>
    </div>

    <h3 class="review__title">Your answers</h3>
    <ol class="review">
      <li v-for="round in rounds" :key="round.question.id" class="review__item">
        <p class="review__question">{{ round.question.question.text }}</p>
        <p v-if="isCorrect(round)" class="review__answer review__answer--correct">
          ✓ {{ round.chosen }} <span class="visually-hidden">(correct)</span>
        </p>
        <template v-else>
          <p class="review__answer review__answer--incorrect">
            ✗ {{ round.chosen }} <span class="visually-hidden">(incorrect)</span>
          </p>
          <p class="review__answer">Answer: {{ round.question.correctAnswer }}</p>
        </template>
      </li>
    </ol>
  </section>
</template>
