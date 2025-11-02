<template>
  {{ String(hh).padStart(2, "0") }}h
  {{ String(mm).padStart(2, "0") }}m
  {{ String(ss).padStart(2, "0") }}s
  {{ String(mis).padStart(3, "0") }}ms
</template>

<script setup>
import { computed } from 'vue'
const props = defineProps(["ms"])
const hms = computed(() => new Date(props.ms - 3600000))
const hh = computed(() => hms.value.getHours())
const mm = computed(() => hms.value.getMinutes())
const ss = computed(() => hms.value.getSeconds())
const mis = computed(() => hms.value.getMilliseconds())
</script>
