<template>
<v-container fluid>
  <v-row justify="center">
    <v-col cols="12" lg="10" xl="10">

      <!-- Header Section -->
      <v-row>
        <v-col cols="12" class="text-left mt-5 mb-2 d-flex align-center">
          <v-icon size="90" class="mr-3 align-self-center">mdi-cube</v-icon>
          <div>
            <h1 class="text-h3 font-weight-bold mb-2">Masterplan Hub</h1>
            <div class="text-subtitle-1 text-grey-darken-1">App Project Management</div>
          </div>
        </v-col>
      </v-row>

      <!-- Project Navigation -->
      <v-row>
        <v-col cols="12">
          <v-card class="mb-0">
            <v-card-title class="bg-grey-darken-4">
              <v-icon start icon="mdi-office-building" class="mr-2"></v-icon>
              Project Selection
            </v-card-title>
            <v-card-text>
              <v-btn-group variant="outlined" class="w-100">
                <v-btn
                  v-for="proj in projects"
                  :key="proj"
                  :to="`/projects/${proj}`"
                  :color="proj === projectId ? 'primary' : undefined"
                  size="large"
                >
                  {{ proj }}
                </v-btn>
              </v-btn-group>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>

      <!-- Main Content Area -->
      <v-row>
        <!-- Project Statistics Card -->
        <v-col cols="12" md="6" lg="3">
          <v-card class="mb-4">
            <v-card-title class="bg-grey-darken-4">
              <v-icon start icon="mdi-chart-box" class="mr-2"></v-icon>
              Project Statistics
            </v-card-title>
            <v-card-text>
              <v-list density="compact">
                <v-list-item>
                  <template v-slot:prepend>
                    <v-icon color="green">mdi-home</v-icon>
                  </template>
                  <v-list-item-title>Available Units</v-list-item-title>
                  <v-list-item-subtitle>{{ filteredUnits.filter(u => u.State === 'Free').length }}</v-list-item-subtitle>
                </v-list-item>
                <v-list-item>
                  <template v-slot:prepend>
                    <v-icon color="orange">mdi-clock</v-icon>
                  </template>
                  <v-list-item-title>Reserved Units</v-list-item-title>
                  <v-list-item-subtitle>{{ filteredUnits.filter(u => u.State === 'Reserved').length }}</v-list-item-subtitle>
                </v-list-item>
                <v-list-item>
                  <template v-slot:prepend>
                    <v-icon color="red">mdi-check-circle</v-icon>
                  </template>
                  <v-list-item-title>Sold Units</v-list-item-title>
                  <v-list-item-subtitle>{{ filteredUnits.filter(u => u.State === 'Sold').length }}</v-list-item-subtitle>
                </v-list-item>
              </v-list>
            </v-card-text>
          </v-card>

          <!-- Filters Card -->
          <v-card>
            <v-card-title class="bg-grey-darken-4">
              <v-icon start icon="mdi-filter" class="mr-2"></v-icon>
              Filters
            </v-card-title>
            <v-card-text>
              <v-select
                v-model="selectedDisposition"
                :items="dispositions"
                label="Disposition"
                clearable
                hide-details="auto"
                class="mb-4"
                density="compact"
              ></v-select>
              <v-select
                v-model="selectedFloor"
                :items="floors"
                label="Floor"
                clearable
                hide-details="auto"
                class="mb-4"
                density="compact"
              ></v-select>
              <v-select
                v-model="selectedState"
                :items="states"
                label="State"
                clearable
                hide-details="auto"
                density="compact"
              ></v-select>
            </v-card-text>
          </v-card>
            
          <!-- Project Log Section -->
          <v-row>
            <v-col cols="12">
              <v-card class="mt-4">
                <v-card-title class="bg-grey-darken-4">
                  <v-icon start icon="mdi-history" class="mr-2"></v-icon>
                  Project Activity Log
                </v-card-title>
                <v-card-text>
                  <ProjectLog
                    ref="logComponent"
                    :project-id="projectId"
                  />
                </v-card-text>
              </v-card>
            </v-col>
          </v-row>

          <!-- Project Activity -->
          <v-row>
            <v-col cols="12">
              <LogActivityChart :project-id="projectId" />
            </v-col>
          </v-row>

        </v-col>

        
        <!-- Main Content -->
        <v-col cols="12" md="6" lg="9">
          <!-- View Toggle -->
          <v-card class="mb-4">
            <v-card-text>
              <v-btn-group>
                <v-btn
                  :color="viewMode === 'card' ? 'primary' : undefined"
                  @click="viewMode = 'card'"
                  prepend-icon="mdi-view-grid"
                >
                  Card View
                </v-btn>
                <v-btn
                  :color="viewMode === 'table' ? 'primary' : undefined"
                  @click="viewMode = 'table'"
                  prepend-icon="mdi-table"
                >
                  Table View
                </v-btn>
              </v-btn-group>
            </v-card-text>
          </v-card>

          <!-- Table View -->
          <v-card v-if="viewMode === 'table'">
            <v-table hover density="compact">
              <thead>
                <tr>
                  <th>Unit</th>
                  <th>Floor</th>
                  <th>Disposition</th>
                  <th>Area (m²)</th>
                  <th>Orientation</th>
                  <th>Price (CZK)</th>
                  <th>Price with VAT (CZK)</th>
                  <th>State</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="unit in filteredUnits" :key="unit.Name">
                  <td>
                    <v-avatar size="32" class="mr-2">
                      <v-img
                        :src="`${baseUrl}/api/projects/${projectId}/images/${unit.Name.toLowerCase()}.svg`"
                        cover
                      ></v-img>
                    </v-avatar>
                    {{ unit.Name }}
                  </td>
                  <td>{{ unit.Floor }}</td>
                  <td>{{ unit.Disposition }}</td>
                  <td>{{ unit.TotalArea }}</td>
                  <td>{{ unit.Orientation }}</td>
                  <td>{{ formatPrice(unit.Price) }}</td>
                  <td>{{ formatPrice(unit.PriceVAT) }}</td>
                  <td>
                    <v-chip
                      :color="getStateColor(unit.State)"
                      size="small"
                    >
                      {{ unit.State }}
                    </v-chip>
                  </td>
                </tr>
              </tbody>
            </v-table>
          </v-card>

          <!-- Card View -->
          <v-row v-else>
            <v-col 
              v-for="unit in filteredUnits" 
              :key="unit.Name"
              cols="12"
              sm="3"
              lg="2"
            >
              <v-card>
                <v-img
                  :src="`${baseUrl}/api/projects/${projectId}/images/${unit.Name.toLowerCase()}.svg`"
                  height="200"
                  cover
                  class="bg-grey-lighten-3"
                >
                  <template v-slot:placeholder>
                    <v-row
                      class="fill-height ma-0"
                      align="center"
                      justify="center"
                    >
                      <v-progress-circular
                        indeterminate
                        color="grey-lighten-5"
                      ></v-progress-circular>
                    </v-row>
                  </template>
                </v-img>
                
                <v-card-title class="d-flex justify-space-between align-center">
                  <span>{{ unit.Name }}</span>
                  <v-chip
                    :color="getStateColor(unit.State)"
                    size="small"
                  >
                    {{ unit.State }}
                  </v-chip>
                </v-card-title>

                <v-card-text>
                  <v-row dense>
                    <v-col cols="6">
                      <div class="text-caption text-grey">Disposition</div>
                      <div class="font-weight-medium">{{ unit.Disposition }}</div>
                    </v-col>
                    <v-col cols="6">
                      <div class="text-caption text-grey">Floor</div>
                      <div class="font-weight-medium">{{ unit.Floor }}</div>
                    </v-col>
                    <v-col cols="6">
                      <div class="text-caption text-grey">Area</div>
                      <div class="font-weight-medium">{{ unit.TotalArea }} m²</div>
                    </v-col>
                    <v-col cols="6">
                      <div class="text-caption text-grey">Orientation</div>
                      <div class="font-weight-medium">{{ unit.Orientation }}</div>
                    </v-col>
                    <v-col cols="12" class="mt-2">
                      <div class="text-caption text-grey">Price</div>
                      <div class="text-h6 font-weight-bold">{{ formatPrice(unit.Price) }} CZK</div>
                      <div class="text-caption">{{ formatPrice(unit.PriceVAT) }} CZK with VAT</div>
                    </v-col>
                  </v-row>
                </v-card-text>
              </v-card>
            </v-col>
          </v-row>
        </v-col>
      </v-row>

      </v-col>
  </v-row>
</v-container>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import { useRoute } from 'vue-router'
import ProjectLog from '@/components/ProjectLog.vue'
import LogActivityChart from '@/components/LogActivityChart.vue'

const baseUrl = import.meta.env.VITE_API_URL || 'https://masterplanhub.pythonanywhere.com'
const route = useRoute()

const units = ref([])
const projects = ref([])
const projectId = computed(() => route.params.id)
const viewMode = ref('card')

const selectedDisposition = ref(null)
const selectedFloor = ref(null)
const selectedState = ref(null)

const dispositions = computed(() => 
  [...new Set(units.value.map(unit => unit.Disposition))].sort()
)

const floors = computed(() => 
  [...new Set(units.value.map(unit => unit.Floor))].sort()
)

const states = computed(() => 
  [...new Set(units.value.map(unit => unit.State))].sort()
)

const filteredUnits = computed(() => 
  units.value.filter(unit => {
    if (selectedDisposition.value && unit.Disposition !== selectedDisposition.value) return false
    if (selectedFloor.value && unit.Floor !== selectedFloor.value) return false
    if (selectedState.value && unit.State !== selectedState.value) return false
    return true
  })
)

const formatPrice = (price) => {
  return new Intl.NumberFormat('cs-CZ').format(price)
}

const getStateColor = (state) => {
  const colors = {
    'Free': 'green',
    'Reserved': 'orange',
    'Sold': 'red',
    'Unknown': 'grey'
  }
  return colors[state] || 'grey'
}

const loadProject = async () => {
  try {
    const response = await fetch(`${baseUrl}/api/projects/${projectId.value}/data`)
    units.value = await response.json()
  } catch (error) {
    console.error('Error loading project:', error)
  }
}

const loadProjects = async () => {
  try {
    const response = await fetch(`${baseUrl}/api/projects`)
    projects.value = await response.json()
  } catch (error) {
    console.error('Error loading projects:', error)
  }
}

watch(() => route.params.id, (newId) => {
  if (newId) {
    loadProject()
  }
})

onMounted(async () => {
  await loadProjects()
  if (projectId.value) {
    await loadProject()
  }
})
</script>