<script setup lang="ts">
import { computed, nextTick, onMounted, useTemplateRef, watch } from "vue";
import { CATEGORY_LABELS } from "../categories";
import type { Round } from "../types";

const props = defineProps<{
  round: Round;
  /** Which question this is, starting from 1 */
  number: number;
  total: number;
  score: number;
}>();
const emit = defineEmits<{ answer: [answer: string]; next: [] }>();

type AnswerState = "idle" | "correct" | "incorrect";

const question = computed(() => props.round.question);
const hasAnswered = computed(() => props.round.chosen !== null);
const isLastQuestion = computed(() => props.number === props.total);
const answeredCount = computed(() => props.number - 1 + (hasAnswered.value ? 1 : 0));

const heading = useTemplateRef("heading");
const nextButton = useTemplateRef("nextButton");

// Keyboard and screen reader users would otherwise be left on a button that
// no longer exists, so move focus to the new question...
onMounted(() => heading.value?.focus());
// ...and to the "next" button once they've answered
watch(hasAnswered, async (answered) => {
  if (!answered) return;
  // Wait for the button to become visible before focusing it
  await nextTick();
  nextButton.value?.focus();
});

const answerState = (answer: string): AnswerState => {
  if (!hasAnswered.value) return "idle";
  if (answer === question.value.correctAnswer) return "correct";
  if (answer === props.round.chosen) return "incorrect";
  return "idle";
};

const choose = (answer: string) => {
  if (!hasAnswered.value) emit("answer", answer);
};
</script>

<template>
  <section class="card" aria-labelledby="question-text">
    <div class="quiz-meta">
      <span>Question {{ number }} of {{ total }}</span>
      <span>Score: {{ score }}</span>
    </div>
    <div
      class="progress"
      role="progressbar"
      aria-label="Questions answered"
      aria-valuemin="0"
      :aria-valuemax="total"
      :aria-valuenow="answeredCount"
    >
      <div class="progress__bar" :style="{ width: `${(answeredCount / total) * 100}%` }" />
    </div>

    <p class="question__category">
      {{ CATEGORY_LABELS[question.category] }} · {{ question.difficulty }}
    </p>
    <h2 id="question-text" ref="heading" tabindex="-1" class="question__text">
      {{ question.question.text }}
    </h2>

    <ul class="answers">
      <li v-for="answer in round.answers" :key="answer">
        <!--
          aria-disabled rather than disabled: a disabled button can't be
          focused, which would throw keyboard users out of the list
        -->
        <button
          type="button"
          :class="['answer', `answer--${answerState(answer)}`]"
          :aria-disabled="hasAnswered"
          @click="choose(answer)"
        >
          <span>{{ answer }}</span>
          <!-- Text as well as colour, so the result doesn't rely on seeing red and green -->
          <span v-if="answerState(answer) === 'correct'" class="answer__tag">Correct answer</span>
          <span v-if="answerState(answer) === 'incorrect'" class="answer__tag">Your answer</span>
        </button>
      </li>
    </ul>

    <!-- Always rendered, so answering doesn't push the page around -->
    <div class="question__footer">
      <p class="feedback" role="status">
        <template v-if="hasAnswered">
          {{
            round.chosen === question.correctAnswer
              ? "Correct!"
              : `Not quite. The answer was ${question.correctAnswer}.`
          }}
        </template>
      </p>
      <!-- Invisible (but still taking up space) until the player answers -->
      <button
        ref="nextButton"
        type="button"
        :class="['button', 'button--primary', { 'is-invisible': !hasAnswered }]"
        @click="emit('next')"
      >
        {{ isLastQuestion ? "See results" : "Next question" }}
      </button>
    </div>
  </section>
</template>
