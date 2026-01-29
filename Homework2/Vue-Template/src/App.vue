<script setup lang="ts">
import Example from './components/Example.vue'
import Notes from './components/Notes.vue'
import NotesWithStore from './components/NotesWithStore.vue'
import ScatterPlot from './components/ScatterPlot.vue'

// Local components are available in the template automatically with `script setup`.
</script>

<!-- Using Vuetify components (Vuetify should be registered in the app entry). -->
<template>
  <VContainer id="main-container" fluid>
    <VRow>
      <VCol cols="12" md="6">
        <VCard variant="outlined" height="500" class="ma-2">
          <VCardTitle class="text-subtitle-1">XY Scatter Plot</VCardTitle>
          <VCardText style="height: 400px;">
            <ScatterPlot />
          </VCardText>
        </VCard>
      </VCol>

      <VCol cols="12" md="6">
        <VCard variant="outlined" height="500" class="ma-2">
          <VCardTitle class="text-subtitle-1">Categorical Bar Chart</VCardTitle>
          <VCardText style="height: 400px;">
            <Example />
          </VCardText>
        </VCard>
      </VCol>
    </VRow>

    <VRow>
      <VCol cols="12">
        <VCard variant="outlined" class="ma-2">
          <Notes msg="Data sourced from demo.csv and demo.json" />
        </VCard>
      </VCol>
    </VRow>
  </VContainer>
</template>

<style scoped>
#main-container {
  min-height: 100vh;
  background-color: #f5f5f5;
}
</style>


