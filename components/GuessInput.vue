<template>
  <div class="guess-input">
    <input
      v-model="guess"
      @keyup.enter="submitGuess"
      placeholder="Entrez le nom du pays"
    />
    <button @click="submitGuess">Deviner</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import '~/assets/styles.css'

const guess = ref('')
const emit = defineEmits(['guess'])

const submitGuess = () => {
  emit('guess', guess.value)
  guess.value = ''
}
</script>
