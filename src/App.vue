<template>
  <div>
    <!-- CREATION DE COURSE -->
    <div v-if="!courseInfo">
      <h2>Créer la course</h2>
      <CreerCourse @createCourse="setupCourse" />
    </div>

    <div v-else>
      <h1>Nom de la Course : {{ courseInfo.nom }}</h1>
      <p>Âge: {{ courseInfo.ageMin }} - {{ courseInfo.ageMax }}</p>
      <p>Temps max: {{ courseInfo.tempsMax }} min</p>
      <p>Participants: {{ tabPers.length }} / {{ courseInfo.nbMax }}</p>

      <!-- Inscription des participants -->
      <h2>Inscription</h2>
      <InscriptionPersonne v-if="etatInscription === 'enCours'" :course="courseInfo"
        :disabled="tabPers.length >= courseInfo.nbMax" @inscription="inscription" />

      <!-- Liste des participants avant course -->
      <div v-if="etatInscription === 'enCours' && tabPers.length > 0">
        <h3>Participants inscrits</h3>
        <ul>
          <li v-for="p in tabPers" :key="p.id">
            {{ p.id }}. {{ p.nom }} {{ p.prenom }} ({{ p.age }} ans)
            <button @click="deletePerson(p.id)">❌</button>
          </li>
        </ul>
      </div>

      <!-- Bouton démarrer / finir inscription -->
      <button v-if="etatInscription === 'enCours'" @click="terminerInscription"
        :disabled="tabPers.length < courseInfo.nbMin || raceStarted">
        Démarrer la course
      </button>

      <!-- Retour inscription -->
      <button v-if="etatInscription === 'termine'" @click="retourInscription" :disabled="raceStarted || raceFinished">
        Retour à l'inscription
      </button>


      <!-- COURSE EN COURS -->
      <div v-if="etatInscription === 'termine'">
        <h2>Course en cours</h2>

        <button @click="startChrono" :disabled="timer !== null || raceStarted || raceFinished">
          Start Chrono
        </button>

        <TempsHms :ms="tps" />

        <h3>Arrivées / Forfait</h3>
        <BoxTousParticipants :tousLesParticipants="tabPers" :raceStarted="raceStarted" :raceFinished="raceFinished"
          @stop-chrono="arriveeUnParticipant" @forfeit-participant="forfeitParticipant" />

        <!-- LEADERBOARD -->
        <h2 v-if="raceFinished && tabPers.length > 0">🏆 Leaderboard</h2>
        <table v-if="raceFinished && tabPers.length > 0" border="1" cellpadding="5">
          <thead>
            <tr>
              <th>#</th>
              <th>Nom</th>
              <th>Prénom</th>
              <th>Âge</th>
              <th>Temps</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(p, index) in sortedParticipants" :key="p.id" :style="{ color: colorRow(p) }">
              <td>{{ index + 1 }}</td>
              <td>{{ p.nom }}</td>
              <td>{{ p.prenom }}</td>
              <td>{{ p.age }}</td>
              <td>
                <span v-if="p.forfeit">FORFAIT</span>
                <span v-else-if="p.temps === null">NON FINI</span>
                <span v-else>{{ formatTime(p.temps) }}</span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Reset Button -->
    <div v-if="raceFinished && tabPers.length > 0" style="margin-top:10px;">
      <button @click="resetRace">
        Réinitialiser la course
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue"
import CreerCourse from "./components/CreerCourse.vue"
import InscriptionPersonne from "./components/InscriptionPersonne.vue"
import BoxTousParticipants from "./components/BoxTousParticipants.vue"
import TempsHms from "./components/TempsHms.vue"

/* État de la course */
const courseInfo = ref(null)
const etatInscription = ref("enCours")
const tabPers = ref([])
let id = 1

const raceStarted = ref(false)
let timer = null
const hDep = ref(null)
const hActuelle = ref(null)

/* COURSE CREATION */
const setupCourse = (course) => {
  courseInfo.value = course
}

/* INSCRIPTION PARTICIPANTS */
const inscription = (pers) => {
  if (tabPers.value.length >= courseInfo.value.nbMax) return
  tabPers.value.push({ ...pers, id: id++, temps: null, forfeit: false })
}

const deletePerson = (pid) => {
  tabPers.value = tabPers.value.filter(p => p.id !== pid)
  if (tabPers.value.length === 0) {
    etatInscription.value = "enCours"
    id = 1
  }
}

const terminerInscription = () => {
  if (tabPers.value.length < courseInfo.value.nbMin) return
  etatInscription.value = "termine"
}

const retourInscription = () => {
  etatInscription.value = "enCours"
  raceStarted.value = false
  clearInterval(timer)
  timer = null
  hDep.value = null
  hActuelle.value = null
  tabPers.value.forEach(p => {
    p.temps = null
    p.forfeit = false
  })
}

/* CHRONO */
const startChrono = () => {
  if (timer) return
  hDep.value = new Date()
  raceStarted.value = true

  timer = setInterval(() => {
    hActuelle.value = new Date()
    const elapsedMs = hActuelle.value - hDep.value
    const maxMs = courseInfo.value.tempsMax * 60 * 1000

    // Mark as forfeit if time exceeded
    if (elapsedMs >= maxMs) {
      tabPers.value.forEach(p => {
        if (p.temps === null && !p.forfeit) p.forfeit = true
      })
      stopChrono()
    }
  }, 10)
}

const tps = computed(() =>
  hActuelle.value && hDep.value ? hActuelle.value - hDep.value : 0
)

/* BTN PARTICIPANT */
const arriveeUnParticipant = (idPart) => {
  if (!raceStarted.value) return
  let p = tabPers.value.find(x => x.id === idPart)
  if (!p || p.temps !== null || p.forfeit) return
  p.temps = tps.value
  stopChrono()
}

const forfeitParticipant = (idPart) => {
  if (!raceStarted.value) return
  let p = tabPers.value.find(x => x.id === idPart)
  if (!p || p.temps !== null || p.forfeit) return
  p.forfeit = true
  stopChrono()
}

const stopChrono = () => {
  const unfinished = tabPers.value.some(p => p.temps === null && !p.forfeit)
  if (!unfinished && timer) {
    clearInterval(timer)
    timer = null
    raceStarted.value = false
  }
}

/* LEADERBOARD */
const sortedParticipants = computed(() => {
  return [...tabPers.value].sort((a, b) => {
    if (a.temps !== null && b.temps !== null) return a.temps - b.temps
    if (a.temps !== null && (b.forfeit || b.temps === null)) return -1
    if ((a.forfeit || a.temps === null) && b.temps !== null) return 1
    if (a.forfeit && b.temps === null) return -1
    if (a.temps === null && b.forfeit) return 1
    return 0
  })
})

const raceFinished = computed(() => {
  return tabPers.value.every(p => p.temps !== null || p.forfeit)
})

/* Couleur leaderboard */
const colorRow = (p) => {
  if (p.temps !== null) return "green"
  if (p.forfeit) return "red"
  return "red"
}

/* temps */
const formatTime = (ms) => {
  const totalSeconds = Math.floor(ms / 1000)
  const minutes = Math.floor(totalSeconds / 60)
  const seconds = totalSeconds % 60
  const msRemaining = ms % 1000
  return `${minutes}m ${seconds}s ${msRemaining}ms`
}

/* RESET COURSE */
const resetRace = () => {
  clearInterval(timer)
  timer = null
  hDep.value = null
  hActuelle.value = null
  raceStarted.value = false
  tabPers.value = []
  etatInscription.value = "enCours"
  courseInfo.value = null
  id = 1
}
</script>

<style scoped>
ul {
  padding: 0;
  list-style: none;
}

li {
  margin-bottom: 4px;
}

button {
  margin-top: 6px;
}

table {
  border-collapse: collapse;
  margin-top: 10px;
}

th,
td {
  padding: 5px 10px;
  text-align: left;
}
</style>
