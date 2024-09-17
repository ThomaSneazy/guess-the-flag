<template>
  <div class="guess-input" ref="guessInputContainer">
    <button 
      v-for="country in countryOptions" 
      :key="country" 
      @click="submitGuess(country)"
      class="country-button"
      :class="{ 
        'incorrect': incorrectGuess === country,
        'correct': correctGuess === country
      }"
    >
      {{ country }}
    </button>
    <button 
      v-if="hintCount > 0"
      @click="showHint" 
      class="hint-button"
    >
      HINT ({{ hintCount }})
    </button>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import gsap from 'gsap'

const props = defineProps({
  correctCountry: { type: String, required: true },
  countries: { type: Array, required: true },
  globalHintCount: { type: Number, required: true }
})

const emit = defineEmits(['guess', 'showHint', 'gameOver', 'updateHintCount'])

const countryOptions = ref([])
const incorrectGuess = ref(null)
const correctGuess = ref(null)
const hasGuessedIncorrectly = ref(false)
const score = ref(0)
const hintCount = ref(props.globalHintCount)
const guessInputContainer = ref(null)

watch(() => props.correctCountry, () => {
  fadeOut().then(() => {
    generateCountryOptions()
    incorrectGuess.value = null
    correctGuess.value = null
    hasGuessedIncorrectly.value = false
    fadeIn()
  })
}, { immediate: true })

watch(() => props.globalHintCount, (newCount) => {
  hintCount.value = newCount
})

function generateCountryOptions() {
  if (!props.correctCountry || props.countries.length < 3) return

  const options = [props.correctCountry]
  const availableCountries = props.countries.filter(c => c.name !== props.correctCountry)

  while (options.length < 3) {
    const randomCountry = availableCountries[Math.floor(Math.random() * availableCountries.length)].name
    if (!options.includes(randomCountry)) {
      options.push(randomCountry)
    }
  }

  countryOptions.value = options.sort(() => Math.random() - 0.5)
}

function submitGuess(country) {
  if (country === props.correctCountry) {
    correctGuess.value = country
    emit('guess', country)
    incorrectGuess.value = null
    hasGuessedIncorrectly.value = false
    score.value++
    setTimeout(() => {
      fadeOut()
    }, 1000)
  } else {
    incorrectGuess.value = country
    if (hasGuessedIncorrectly.value) {
      emit('gameOver', score.value)
      score.value = 0
    } else {
      hasGuessedIncorrectly.value = true
    }
  }
}

function showHint() {
  if (hintCount.value > 0) {
    hintCount.value--
    emit('updateHintCount', hintCount.value)
    emit('showHint')
  }
}

function fadeOut() {
  return gsap.to(guessInputContainer.value, {
    opacity: 0,
    duration: 0.5
  })
}

function fadeIn() {
  gsap.fromTo(guessInputContainer.value, 
    { opacity: 0 },
    { opacity: 1, duration: 0.5, delay: 1 }
  )
}
</script>

<style scoped>
.guess-input {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  justify-content: center;
  margin-top: 1rem;
}

.country-button, .hint-button {
  padding: 0.5rem 1rem;
  font-size: 1rem;
  cursor: pointer;
  border: none;
  border-radius: 4px;
  transition: background-color 0.3s;
}

.country-button {
  background-color: #ffffff;
  color: rgb(0, 0, 0);
  text-transform: uppercase;
}

.country-button.incorrect {
  background-color: #FF0000;
}

.country-button.correct {
  background-color: #4CAF50;
}

.hint-button {
  background-color: #f1c40f;
  color: black;
}
</style>
