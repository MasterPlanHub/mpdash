<script setup>
import { onMounted } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()

onMounted(async () => {
  try {
    const response = await fetch('http://localhost:5000/api/projects')
    const projects = await response.json()
    if (projects.length > 0) {
      router.replace(`/projects/${projects[0]}`)
    }
  } catch (error) {
    console.error('Error loading projects:', error)
  }
})
</script>

<template>
  <v-container class="fill-height">
    <v-row align="center" justify="center">
      <v-col class="text-center">
        <v-progress-circular
          indeterminate
          color="primary"
        ></v-progress-circular>
      </v-col>
    </v-row>
  </v-container>
</template>