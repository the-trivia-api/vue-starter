<script setup lang="ts">
import { ref } from "vue";
import { describeError, fetchQuestions, type QuizSettings } from "./api";
import { shuffle } from "./shuffle";
import { isCorrect, type Round } from "./types";
import SetupForm from "./components/SetupForm.vue";
import QuestionCard from "./components/QuestionCard.vue";
import Results from "./components/Results.vue";
import StatusCard from "./components/StatusCard.vue";

/**
 * Everything the quiz can be doing. Each status only has the data it needs,
 * so e.g. there's no way to be "playing" without any questions.
 */
type GameState =
  | { status: "setup" }
  | { status: "loading" }
  | { status: "error"; message: string }
  | { status: "playing"; rounds: Round[]; current: number }
  | { status: "finished"; rounds: Round[] };

const settings = ref<QuizSettings>({ category: "any", difficulty: "any", amount: 10 });
const game = ref<GameState>({ status: "setup" });

const startQuiz = async (newSettings: QuizSettings) => {
  settings.value = newSettings;
  game.value = { status: "loading" };

  try {
    const questions = await fetchQuestions(newSettings);
    if (questions.length === 0) {
      game.value = {
        status: "error",
        message: "No questions matched those settings. Try a different category or difficulty.",
      };
      return;
    }

    game.value = {
      status: "playing",
      current: 0,
      rounds: questions.map((question) => ({
        question,
        answers: shuffle([question.correctAnswer, ...question.incorrectAnswers]),
        chosen: null,
      })),
    };
  } catch (error) {
    game.value = { status: "error", message: describeError(error) };
  }
};

const answer = (choice: string) => {
  if (game.value.status !== "playing") return;
  game.value.rounds[game.value.current].chosen = choice;
};

const goToNextQuestion = () => {
  if (game.value.status !== "playing") return;
  if (game.value.current + 1 < game.value.rounds.length) {
    game.value.current++;
  } else {
    game.value = { status: "finished", rounds: game.value.rounds };
  }
};

const changeSettings = () => {
  game.value = { status: "setup" };
};
</script>

<template>
  <div class="page">
    <header class="header">
      <h1 class="header__title">Trivia Quiz</h1>
      <p class="header__subtitle">
        Built with Vue and
        <a href="https://the-trivia-api.com" class="link">The Trivia API</a>
      </p>
    </header>

    <main>
      <SetupForm
        v-if="game.status === 'setup'"
        :default-settings="settings"
        @start="startQuiz"
      />

      <StatusCard v-else-if="game.status === 'loading'" title="Loading questions…" is-loading />

      <StatusCard
        v-else-if="game.status === 'error'"
        title="Something went wrong"
        :message="game.message"
        @retry="startQuiz(settings)"
        @change-settings="changeSettings"
      />

      <!-- A new key per question resets the card's focus handling -->
      <QuestionCard
        v-else-if="game.status === 'playing'"
        :key="game.rounds[game.current].question.id"
        :round="game.rounds[game.current]"
        :number="game.current + 1"
        :total="game.rounds.length"
        :score="game.rounds.filter(isCorrect).length"
        @answer="answer"
        @next="goToNextQuestion"
      />

      <Results
        v-else-if="game.status === 'finished'"
        :rounds="game.rounds"
        @play-again="startQuiz(settings)"
        @change-settings="changeSettings"
      />
    </main>

    <footer class="footer">
      <a href="https://github.com/the-trivia-api/vue-starter" class="link">
        View the code on GitHub
      </a>
      <span aria-hidden="true">·</span>
      <a href="https://the-trivia-api.com/docs/" class="link">API docs</a>
    </footer>
  </div>
</template>
