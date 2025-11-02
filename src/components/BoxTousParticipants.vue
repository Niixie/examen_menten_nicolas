<template>
  <div>
    <BoxUnParticipant v-for="p in tousLesParticipants" :key="p.id" :unParticipant="p" :raceStarted="raceStarted"
      @participant-arrive="$emit('stop-chrono', p.id)" @forfeit-participant="$emit('forfeit-participant', p.id)" />
  </div>
</template>

<script setup>
import BoxUnParticipant from './BoxUnParticipant.vue'

defineProps(["tousLesParticipants", "raceStarted"])
defineEmits(['stop-chrono', 'forfeit-participant'])
</script>

<style scoped>
div {
  margin-top: 10px;
}
</style>
