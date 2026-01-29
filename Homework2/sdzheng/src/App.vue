<script setup lang="ts">
import Example from './components/Example.vue'
import Notes from './components/Notes.vue'
import NotesWithStore from './components/NotesWithStore.vue'
import ScatterPlot from './components/ScatterPlot.vue'
import Histogram from './components/Histogram.vue'
import GenrePieChart from './components/GenrePieChart.vue'
import ParallelCoordinatesPlot from './components/ParallelCoordinatesPlot.vue'

// Local components are available in the template automatically with `script setup`.
</script>

<!-- Using Vuetify components (Vuetify should be registered in the app entry). -->
<template>
  <VContainer id="main-container" fluid>
    <VRow>
      <VCol cols="12">
        <VCard variant="outlined" height="600" class="ma-2">
          <VCardTitle class="text-h6 font-weight-bold">
            1. How to Write a Popular Book: Book Metrics That Affect Popularity
          </VCardTitle>
          <VCardText style="height: 500px;">
            <ParallelCoordinatesPlot />
          </VCardText>
        </VCard>
      </VCol>
    </VRow>

    <VRow>
      <VCol cols="12" md="6">
        <VCard variant="outlined" height="600" class="ma-2">
          <VCardTitle class="text-h6 font-weight-bold">
            2. Page Count Distribution
          </VCardTitle>
          <VCardText style="height: 500px;">
            <Histogram />
          </VCardText>
        </VCard>
      </VCol>

      <VCol cols="12" md="6">
        <VCard variant="outlined" height="600" class="ma-2">
          <VCardTitle class="text-h6 font-weight-bold">
            3. Genre Breakdown (300-350 Pages)
          </VCardTitle>
          <VCardText style="height: 500px;">
            <GenrePieChart />
          </VCardText>
        </VCard>
      </VCol>
    </VRow>

    
  </VContainer>
</template>

<style scoped>
#main-container {
  min-height: 100vh;
  background-color: #fafafa;
  padding: 24px;
}
</style>


