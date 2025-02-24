<template>
    <v-card>
      <v-card-title class="bg-grey-darken-4">
        <v-icon start icon="mdi-chart-line" class="mr-2"></v-icon>
        Activity Timeline
      </v-card-title>
      <v-card-text style="height: 400px;">
        <v-chart class="chart" :option="chartOption" autoresize />
      </v-card-text>
    </v-card>
  </template>
  
  <script setup>
  import { ref, onMounted, watch } from 'vue'
  import VChart from 'vue-echarts'
  import { use } from 'echarts/core'
  import { CanvasRenderer } from 'echarts/renderers'
  import { LineChart } from 'echarts/charts'
  import {
    GridComponent,
    TooltipComponent,
    LegendComponent,
    DataZoomComponent
  } from 'echarts/components'
  
  // Register ECharts components
  use([
    CanvasRenderer,
    LineChart,
    GridComponent,
    TooltipComponent,
    LegendComponent,
    DataZoomComponent
  ])
  
  const props = defineProps({
    projectId: String
  })
  
  const chartOption = ref({
    tooltip: {
      trigger: 'axis',
      formatter: '{b}: {c} activities'
    },
    grid: {
      left: '3%',
      right: '4%',
      bottom: '10%',
      top: '3%',
      containLabel: true
    },
    xAxis: {
      type: 'category',
      data: [],
      axisLabel: {
        rotate: 45
      }
    },
    yAxis: {
      type: 'value',
      name: 'Activities',
      minInterval: 1
    },
    series: [
      {
        name: 'Activity Count',
        type: 'line',
        data: [],
        smooth: true,
        symbol: 'circle',
        symbolSize: 8,
        itemStyle: {
          color: '#7C4DFF'
        },
        lineStyle: {
          width: 3
        },
        areaStyle: {
          color: {
            type: 'linear',
            x: 0,
            y: 0,
            x2: 0,
            y2: 1,
            colorStops: [
              { offset: 0, color: 'rgba(124,77,255,0.3)' },
              { offset: 1, color: 'rgba(124,77,255,0)' }
            ]
          }
        }
      }
    ],
    dataZoom: [
      {
        type: 'inside',
        start: 0,
        end: 100
      }
    ]
  })
  
  const baseUrl = import.meta.env.VITE_API_URL || 'http://localhost:5000'
  
  const loadAndProcessLogs = async () => {
    try {
      const response = await fetch(`${baseUrl}/api/projects/${props.projectId}/log`)
      const logs = await response.json()
      
      // Filter normal logs and group by date
      const normalLogs = logs.filter(log => log.includes('[normal]'))
      const activityByDate = {}
      
      normalLogs.forEach(log => {
        const date = log.split(' ')[0]
        activityByDate[date] = (activityByDate[date] || 0) + 1
      })
      
      // Sort dates and prepare chart data
      const sortedDates = Object.keys(activityByDate).sort()
      const activityCounts = sortedDates.map(date => activityByDate[date])
      
      // Update chart options
      chartOption.value.xAxis.data = sortedDates
      chartOption.value.series[0].data = activityCounts
    } catch (error) {
      console.error('Error loading log activity:', error)
    }
  }
  
  watch(() => props.projectId, (newId) => {
    if (newId) {
      loadAndProcessLogs()
    }
  })
  
  onMounted(() => {
    if (props.projectId) {
      loadAndProcessLogs()
    }
  })
  </script>
  
  <style scoped>
  .chart {
    height: 100%;
    width: 100%;
  }
  </style>