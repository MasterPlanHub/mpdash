<template>
  <v-card class="fill-height">
    <v-card-title class="d-flex align-center py-2">
      <span class="text-subtitle-2">Filter messages</span>
      <v-spacer></v-spacer>
      
      <v-btn-toggle
        v-model="showMode"
        mandatory
        class="me-2"
        density="compact"
      >
        <v-btn
          value="all"
          size="x-small"
          class="text-caption"
        >
          All
        </v-btn>
        <v-btn
          value="normal"
          size="x-small"
          class="text-caption"
        >
          Normal
        </v-btn>
        <v-btn
          value="dev"
          size="x-small"
          class="text-caption"
        >
          Dev
        </v-btn>
      </v-btn-toggle>
      
      <v-btn
        icon="mdi-refresh"
        variant="text"
        size="small"
        @click="loadLogs"
      ></v-btn>
    </v-card-title>
    
    <v-card-text
      class="pa-0"
      style="height: 350px;"
      ref="scrollContainer"
    >
      <v-virtual-scroll
        ref="logList"
        :items="reversedFilteredLogs"
        height="350"
        item-height="32"
      >
        <template v-slot:default="{ item }">
          <v-list-item
            :class="{
              'bg-grey-darken-4': isNormalLog(item),
              'bg-deep-purple-darken-4': isDevLog(item),
              'py-1': true
            }"
            density="compact"
          >
            <template v-slot:subtitle>
              <span class="text-caption">{{ item }}</span>
            </template>
          </v-list-item>
        </template>
      </v-virtual-scroll>
    </v-card-text>
  </v-card>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'

const props = defineProps({
  projectId: String
})

const logs = ref([])
const showMode = ref('all')
const logList = ref(null)
const scrollContainer = ref(null)
const baseUrl = import.meta.env.VITE_API_URL || 'http://localhost:5000'

const isDevLog = (log) => log.includes('[dev]')
const isNormalLog = (log) => log.includes('[normal]')

const filteredLogs = computed(() => {
  if (showMode.value === 'all') return logs.value
  if (showMode.value === 'dev') return logs.value.filter(isDevLog)
  if (showMode.value === 'normal') return logs.value.filter(isNormalLog)
  return logs.value
})

const reversedFilteredLogs = computed(() => {
  return [...filteredLogs.value].reverse()
})

const loadLogs = async () => {
  try {
    const response = await fetch(`${baseUrl}/api/projects/${props.projectId}/log`)
    logs.value = await response.json()
  } catch (error) {
    console.error('Error loading logs:', error)
  }
}

watch(() => props.projectId, (newProjectId) => {
  if (newProjectId) {
    loadLogs()
  }
})

onMounted(() => {
  if (props.projectId) {
    loadLogs()
  }
})
</script>