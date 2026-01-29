<script setup lang="ts">
import * as d3 from 'd3'
import { isEmpty, debounce } from 'lodash'
import { ref, watch, onMounted, onBeforeUnmount } from 'vue'

// Local types
interface ComponentSize { width: number; height: number }
interface Margin { left: number; right: number; top: number; bottom: number }
interface CategoricalPoint { category: string; value: number }

const points = ref<CategoricalPoint[]>([])
const size = ref<ComponentSize>({ width: 0, height: 0 })
const margin: Margin = { left: 50, right: 20, top: 20, bottom: 80 }
const chartContainer = ref<HTMLElement | null>(null)

async function read() {
    try {
        // Correct path for your Homework2 structure
        const dataPath = new URL('../../data/demo.csv', import.meta.url).href;
        const readFromCSV = await d3.csv(dataPath, (d) => {
            return { category: d.category as string, value: +d.value! }
        })
        points.value = readFromCSV as CategoricalPoint[]
    } catch (e) {
        console.error("CSV Load Error:", e)
    }
}

function onResize() {
    if (!chartContainer.value) return
    size.value = { 
        width: chartContainer.value.clientWidth, 
        height: chartContainer.value.clientHeight 
    }
}

function initChart() {
    const svg = d3.select('#scatter-svg')
    svg.selectAll('*').remove()

    const xCategories = points.value.map(d => d.category)
    const yMax = d3.max(points.value, d => d.value) || 100

    const xScale = d3.scaleBand<string>()
        .rangeRound([margin.left, size.value.width - margin.right])
        .domain(xCategories)
        .padding(0.1)

    const yScale = d3.scaleLinear()
        .range([size.value.height - margin.bottom, margin.top])
        .domain([0, yMax])

    svg.append('g')
        .attr('transform', `translate(0, ${size.value.height - margin.bottom})`)
        .call(d3.axisBottom(xScale))

    svg.append('g')
        .attr('transform', `translate(${margin.left}, 0)`)
        .call(d3.axisLeft(yScale))

    svg.append('g')
        .selectAll('circle')
        .data(points.value)
        .join('circle')
        .attr('cx', d => (xScale(d.category) || 0) + xScale.bandwidth() / 2)
        .attr('cy', d => yScale(d.value))
        .attr('r', 6)
        .attr('fill', 'steelblue')
}

watch([points, size], ([pVal, sVal]) => {
    if (!isEmpty(pVal) && sVal.width > 0 && sVal.height > 0) {
        initChart()
    }
}, { deep: true })

const debouncedOnResize = debounce(onResize, 100)

onMounted(async () => {
    window.addEventListener('resize', debouncedOnResize)
    await read()
    onResize()
})

onBeforeUnmount(() => {
    window.removeEventListener('resize', debouncedOnResize)
})
</script>

<template>
    <div class="chart-wrapper" ref="chartContainer" style="width: 100%; height: 100%; min-height: 400px;">
        <svg id="scatter-svg" width="100%" height="100%"></svg>
    </div>
</template>

<style scoped>
.chart-wrapper {
    display: flex;
}
</style>