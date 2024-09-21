<template>
  <div class="container">
    <Background3D />
    <div class="hint-container" :class="{ 'show': showHint }">
      <p>{{ currentCountry.hint }}</p>
    </div>
    <FlagDisplay 
      v-if="currentCountry.code && !gameOver" 
      :countryCode="currentCountry.code" 
      :animate="flagAnimation"
    />
    <div class="content">
      <h1>GUESS THE FLAG</h1>
      <GuessInput 
        v-if="currentCountry.name && !gameOver"
        :correctCountry="currentCountry.name"
        :countries="countries"
        :globalHintCount="globalHintCount"
        @guess="checkGuess"
        @showHint="toggleHint"
        @gameOver="handleGameOver"
        @updateHintCount="updateGlobalHintCount"
      />
      <p v-if="message">{{ message }}</p>
      <p v-if="gameOver" class="score">Bravo ! Ton score est de {{ score }} pays deviné{{ score > 1 ? 's' : '' }} d'affilée.</p>
      <button v-if="gameOver" @click="restartGame" class="restart-button">Recommencer</button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import FlagDisplay from '~/components/FlagDisplay.vue'
import GuessInput from '~/components/GuessInput.vue'
import Background3D from '~/components/Background3D.vue'
import '~/assets/styles.css'

const countries = ref([])
const currentCountry = ref({})
const message = ref('')
const flagAnimation = ref('none')
const showHint = ref(false)
const gameOver = ref(false)
const score = ref(0)
const globalHintCount = ref(3)

const loadCountries = async () => {
  console.log('Chargement des pays...')
  try {
    const response = await fetch('https://restcountries.com/v3.1/all')
    const data = await response.json()
    countries.value = data.map(country => ({
      name: country.name.common.toLowerCase(),
      code: country.cca2,
      hint: `Ce pays a une population d'environ ${country.population.toLocaleString()} habitants et sa capitale est ${country.capital?.[0] || 'inconnue'}.`
    }))
    console.log('Pays chargés :', countries.value.length)
    // Sélectionner immédiatement un pays après le chargement
    currentCountry.value = countries.value[Math.floor(Math.random() * countries.value.length)]
    console.log('Pays sélectionné :', currentCountry.value.name)
    // Attendre un court instant avant de déclencher l'animation
    setTimeout(() => {
      flagAnimation.value = 'in'
    }, 100)
  } catch (error) {
    console.error('Erreur lors du chargement des pays :', error)
  }
}

const selectRandomCountry = () => {
  if (countries.value.length > 0) {
    showHint.value = false // Cacher l'indice à chaque transition
    flagAnimation.value = 'out'
    setTimeout(() => {
      currentCountry.value = countries.value[Math.floor(Math.random() * countries.value.length)]
      console.log('Pays sélectionné :', currentCountry.value.name)
      flagAnimation.value = 'in'
    }, 0) // Ajustez ce délai selon vos besoins
  } else {
    console.warn('Aucun pays disponible pour la sélection')
  }
}

const checkGuess = (guess) => {
  if (guess.toLowerCase() === currentCountry.value.name.toLowerCase()) {
    // message.value = 'Correct ! Bien joué !'
    score.value++
    setTimeout(() => {
      // message.value = ''
      selectRandomCountry()
    }, 0)
  }
}

const handleGameOver = (finalScore) => {
  gameOver.value = true
  score.value = finalScore
  message.value = 'Game Over ! Vous avez fait deux erreurs consécutives.'
}

const restartGame = () => {
  gameOver.value = false
  message.value = ''
  score.value = 0
  globalHintCount.value = 3 // Réinitialisation des HINT
  selectRandomCountry()
}

const toggleHint = () => {
  showHint.value = !showHint.value
}

const updateGlobalHintCount = (newCount) => {
  globalHintCount.value = newCount
}

onMounted(() => {
  console.log('Composant monté')
  loadCountries()
})
</script>

<style scoped>
.container {
  position: relative;
  width: 100%;
  min-height: 100vh;
  overflow: visible;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.content {
  width: 100%;
  max-width: 70rem;
  position: relative;
  padding: 1rem;
  z-index: 3;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: end;
  height: 90vh;
}

.input-container {
  display: flex;
  gap: 1rem;
  margin-top: 1rem;
  width: 100%;
}

/* .input{
  color: red;
  border: none;
  border-radius: 10rem;
} */

/* .hint-button {
  padding: 0.5rem 1rem;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
} */

.hint-container {
  position: absolute;
  left: 1rem;
  top: 1rem;
  width: 300px;
  background-color: rgba(0, 0, 0, 0.8);
  color: white;
  padding: 1rem;
  border-radius: 4px;
  opacity: 0;
  transition: opacity 0.3s ease-in-out;
}

.hint-container.show {
  opacity: 1;
}

.restart-button {
  margin-top: 1rem;
  padding: 0.5rem 1rem;
  font-size: 1rem;
  cursor: pointer;
  background-color: #3498db;
  color: white;
  border: none;
  border-radius: 4px;
  height: 3rem;
}

.score {
  font-size: 1.2rem;
  font-weight: bold;
  margin-top: 1rem;
}
</style>

