<script setup lang="ts">
import Calendar from '@/components/Calendar.vue'
</script>

<template>
  <calendar init-date="2025-08-13" @get-date="(value) => console.log(value)" />
</template>

<style scoped></style>
