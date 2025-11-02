<template>
  <div>
    <p>Nom : <input type="text" v-model="pers.nom"></p>
    <p>Prénom : <input type="text" v-model="pers.prenom"></p>
    <p>Âge : <input type="number" v-model.number="pers.age"></p>

    <button @click="inscrire" :disabled="disabled">Inscrire</button>
  </div>
</template>

<script setup>
import { ref } from "vue"

const props = defineProps(["course", "disabled"])
const emit = defineEmits(["inscription"])
const pers = ref({ nom: "", prenom: "", age: null })

const inscrire = () => {
  if (!pers.value.nom.trim()) return
  if (!pers.value.prenom.trim()) return
  if (!pers.value.age) return
  if (isNaN(pers.value.age)) return
  if (pers.value.age < props.course.ageMin || pers.value.age > props.course.ageMax) return

  emit("inscription", { ...pers.value })
  pers.value = { nom: "", prenom: "", age: null }
}
</script>
