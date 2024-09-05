<template>
  <div class="container">
    <Background3D />
    <FlagDisplay 
      v-if="currentCountry.code" 
      :countryCode="currentCountry.code" 
      :animate="flagAnimation"
    />
    <div class="content">
      <h1>GUESS THE FLAG</h1>
      <GuessInput @guess="checkGuess" />
      <p v-if="message">{{ message }}</p>
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

const loadCountries = async () => {
  console.log('Chargement des pays...')
  try {
    const response = await fetch('https://restcountries.com/v3.1/all')
    const data = await response.json()
    countries.value = data.map(country => ({
      name: country.name.common.toLowerCase(),
      code: country.cca2
    }))
    console.log('Pays chargés :', countries.value.length)
    // Sélectionner immédiatement un pays après le chargement
    currentCountry.value = countries.value[Math.floor(Math.random() * countries.value.length)]
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
    if (currentCountry.value.code) {
      flagAnimation.value = 'out'
      setTimeout(() => {
        currentCountry.value = countries.value[Math.floor(Math.random() * countries.value.length)]
        console.log('Pays sélectionné :', currentCountry.value)
        flagAnimation.value = 'in'
      }, 1000) // Augmenté à 1000ms pour laisser plus de temps à l'animation de sortie
    } else {
      currentCountry.value = countries.value[Math.floor(Math.random() * countries.value.length)]
      console.log('Premier pays sélectionné :', currentCountry.value)
      flagAnimation.value = 'in'
    }
  } else {
    console.warn('Aucun pays disponible pour la sélection')
  }
}

const checkGuess = (guess) => {
  console.log('Tentative de deviner :', guess)
  if (guess.toLowerCase() === currentCountry.value.name) {
    message.value = 'Correct ! Bien joué !'
    console.log('Réponse correcte')
    setTimeout(() => {
      message.value = ''
      selectRandomCountry()
    }, 2000)
  } else {
    message.value = 'Incorrect. Essayez encore !'
    console.log('Réponse incorrecte')
  }
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
  justify-content: space-between;
}

.content {
  margin: 0 auto;
  max-width: 70rem;
  position: relative;
  padding: 1rem;
  z-index: 3;
  margin-top: 80vh;
}
</style>

