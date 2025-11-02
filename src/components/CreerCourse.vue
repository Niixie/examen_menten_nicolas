<template>
  <div>
    <p>Nom de la course :
      <input type="text" v-model="course.nom">
    </p>

    <p>Âge minimum :
      <input type="number" v-model.number="course.ageMin" min="1">
    </p>

    <p>Âge maximum :
      <input type="number" v-model.number="course.ageMax">
    </p>

    <p>Temps maximum (minutes) :
      <input type="number" v-model.number="course.tempsMax" min="1">
    </p>
    <p>Nombre minimum concurrents :
      <input type="number" v-model.number="course.nbMin" min="2">
    </p>

    <p>Nombre maximum concurrents :
      <input type="number" v-model.number="course.nbMax">
    </p>


    <button @click="creerCourse">Créer la course</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const emit = defineEmits(["createCourse"])

const course = ref({
  nom: "",
  ageMin: 1,
  ageMax: 99,
  tempsMax: 1,
  nbMin: 2,
  nbMax: 50
})

const creerCourse = () => {
  if (course.value.nom.trim() === "") return
  if (course.value.ageMin <= 0) return
  if (course.value.ageMax <= course.value.ageMin) return
  if (course.value.tempsMax <= 0) return
  if (course.value.nbMin < 2) return
  if (course.value.nbMax < course.value.nbMin) return


  emit("createCourse", { ...course.value })
  course.value = { nom: "", ageMin: 1, ageMax: 99, tempsMax: 1, nbMin: 2, nbMax: 50 }
}

</script>
