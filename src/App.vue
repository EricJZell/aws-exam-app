<script setup>
import { ref, onMounted } from 'vue'
import questions from './aws_developer_associate_questions.json'

const examStarted = ref(false)
const currentQuestionIndex = ref(0)
const userAnswers = ref({})
const showResults = ref(false)
const timer = ref(0)
const timerInterval = ref(null)

const startExam = () => {
  examStarted.value = true
  startTimer()
}

const startTimer = () => {
  timerInterval.value = setInterval(() => {
    timer.value++
  }, 1000)
}

const formatTime = (seconds) => {
  const minutes = Math.floor(seconds / 60)
  const remainingSeconds = seconds % 60
  return `${minutes}:${remainingSeconds.toString().padStart(2, '0')}`
}

const nextQuestion = () => {
  if (currentQuestionIndex.value < questions.length - 1) {
    currentQuestionIndex.value++
  }
}

const previousQuestion = () => {
  if (currentQuestionIndex.value > 0) {
    currentQuestionIndex.value--
  }
}

const submitExam = () => {
  clearInterval(timerInterval.value)
  showResults.value = true
}

const calculateScore = () => {
  let correct = 0
  Object.keys(userAnswers.value).forEach(index => {
    if (userAnswers.value[index] === questions[index].correctAnswer) {
      correct++
    }
  })
  return {
    correct,
    total: questions.length,
    percentage: Math.round((correct / questions.length) * 100)
  }
}
</script>

<template>
  <div class="exam-app">
    <template v-if="!examStarted">
      <div class="welcome-screen">
        <h1>AWS Developer Associate Practice Exam</h1>
        <p>Test your knowledge with our practice questions</p>
        <button @click="startExam" class="primary-button">Begin Exam</button>
      </div>
    </template>

    <template v-else-if="!showResults">
      <div class="exam-header">
        <div class="timer">Time: {{ formatTime(timer) }}</div>
        <div class="progress">Question {{ currentQuestionIndex + 1 }} of {{ questions.length }}</div>
      </div>

      <div class="question-container">
        <p class="question">{{ questions[currentQuestionIndex].question }}</p>

        <div class="options">
          <label v-for="(option, index) in questions[currentQuestionIndex].options" :key="index" class="option">
            <input
              type="radio"
              :name="'question-' + currentQuestionIndex"
              :value="option"
              v-model="userAnswers[currentQuestionIndex]"
            >
            <span class="option-text">{{ option }}</span>
          </label>
        </div>

        <div class="navigation">
          <button @click="previousQuestion" :disabled="currentQuestionIndex === 0" class="nav-button">
            ←
          </button>
          <button
            v-if="currentQuestionIndex === questions.length - 1 && Object.keys(userAnswers).length === questions.length"
            @click="submitExam"
            class="primary-button"
          >
            Submit
          </button>
          <button
            v-else
            @click="nextQuestion"
            :disabled="currentQuestionIndex === questions.length - 1"
            class="nav-button"
          >
            →
          </button>
        </div>
      </div>
    </template>

    <template v-else>
      <div class="results">
        <h2>Exam Results</h2>
        <div class="score">
          <p>Score: {{ calculateScore().correct }} / {{ calculateScore().total }}</p>
          <p>Percentage: {{ calculateScore().percentage }}%</p>
          <p>Time taken: {{ formatTime(timer) }}</p>
        </div>
        <div class="review">
          <div v-for="(question, index) in questions" :key="index" class="question-review">
            <p class="category">Category: {{ question.category }}</p>
            <p class="question">{{ question.question }}</p>
            <p class="user-answer" :class="{ correct: userAnswers[index] === question.correctAnswer }">
              Your answer: {{ userAnswers[index] || 'Not answered' }}
            </p>
            <p class="correct-answer">Correct answer: {{ question.correctAnswer }}</p>
            <p class="explanation">{{ question.explanation }}</p>
          </div>
        </div>
        <button @click="examStarted = false; showResults = false; currentQuestionIndex = 0; userAnswers = {}; timer = 0" class="primary-button">
          Start New Exam
        </button>
      </div>
    </template>
  </div>
</template>

<style>
:root {
  --text-color: #111111;
  --accent-color: #2945EB;
  --background-color: #FFFFFF;
}

body {
  margin: 0;
  background-color: var(--background-color);
  color: var(--text-color);
  font-family: system-ui, -apple-system, sans-serif;
}

.exam-app {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
}

.welcome-screen {
  text-align: center;
  padding: 4rem 1rem;
}

.primary-button {
  background-color: var(--accent-color);
  color: white;
  border: none;
  padding: 1rem 2rem;
  border-radius: 8px;
  font-size: 1.1rem;
  cursor: pointer;
  transition: opacity 0.2s;
}

.primary-button:hover {
  opacity: 0.9;
}

.exam-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 2rem;
  font-size: 1.1rem;
}

.question-container {
  background: white;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.question {
  font-size: 1.2rem;
  margin-bottom: 2rem;
}

.options {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.option {
  display: flex;
  align-items: center;
  padding: 1rem;
  border: 1px solid #ddd;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s;
}

.option:hover {
  border-color: var(--accent-color);
}

.option input[type="radio"] {
  margin-right: 1rem;
}

.navigation {
  display: flex;
  justify-content: space-between;
  margin-top: 2rem;
}

.nav-button {
  background: none;
  border: 1px solid #ddd;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  cursor: pointer;
  font-size: 1.2rem;
}

.nav-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.results-modal {
  background: white;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  text-align: center;
}

.score {
  font-size: 1.2rem;
  line-height: 1.6;
}

@media (max-width: 600px) {
  .exam-app {
    padding: 1rem;
  }

  .question-container {
    padding: 1rem;
  }

  .exam-header {
    flex-direction: column;
    gap: 0.5rem;
    text-align: center;
  }
}
</style>
