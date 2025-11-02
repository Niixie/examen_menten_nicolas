<template>
  <div :style="{ marginBottom: '6px' }">
    {{ unParticipant.id }}. {{ unParticipant.nom }} {{ unParticipant.prenom }} ({{ unParticipant.age }}ans)


    <button @click="$emit('participant-arrive', unParticipant.id)"
      :disabled="!raceStarted || unParticipant.temps !== null || unParticipant.forfeit">
      Stop
    </button>


    <button @click="$emit('forfeit-participant', unParticipant.id)"
      :disabled="!raceStarted || unParticipant.temps !== null || unParticipant.forfeit">
      Forfeit
    </button>

    <!-- Timer or status -->
    <span v-if="unParticipant.temps !== null" style="color:green">
      {{ formatTime(unParticipant.temps) }}
    </span>
    <span v-else-if="unParticipant.forfeit" style="color:red">
      FORFAIT
    </span>
    <span v-else-if="raceStarted" style="color:red">
      NON FINI
    </span>
  </div>
</template>

<script setup>
import { defineProps } from "vue"
defineProps(["unParticipant", "showDelete", "raceStarted"])


const formatTime = (ms) => {
  const totalSeconds = Math.floor(ms / 1000)
  const minutes = Math.floor(totalSeconds / 60)
  const seconds = totalSeconds % 60
  const msRemaining = ms % 1000
  return `${minutes}m ${seconds}s ${msRemaining}ms`
}
</script>

<style scoped>
button {
  margin-left: 6px;
}
</style>
