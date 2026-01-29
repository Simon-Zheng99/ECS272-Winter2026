<script setup lang="ts">
import * as d3 from 'd3'
import { isEmpty, debounce } from 'lodash'
import { ref, watch, onMounted, onBeforeUnmount } from 'vue'

interface BookData {
    pageCount: number;
    rating_average: number;
    title: string;
}

const books = ref<BookData[]>([])
const size = ref({ width: 0, height: 0 })
const margin = { left: 60, right: 30, top: 40, bottom: 60 }
const chartContainer = ref<HTMLElement | null>(null)

async function read() {
    try {
        const dataPath = new URL('../../data/top_1000_most_swapped_books.csv', import.meta.url).href; 
        const data = await d3.csv(dataPath, (d) => {
            return {
                // Ensure these names match your CSV headers exactly
                pageCount: +d.pageCount!, 
                rating_average: +d.rating_average!,
                title: d.title
            }
        })
        books.value = data as BookData[]
    } catch (e) {
        console.error("Error loading book data:", e)
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

    // 1. Create Linear Scales for both X and Y
    const xScale = d3.scaleLinear()
        .domain([0, d3.max(books.value, d => d.pageCount) || 1000])
        .range([margin.left, size.value.width - margin.right])
        .nice()

    const yScale = d3.scaleLinear()
        .domain([0, 5]) // Ratings are usually 0 to 5
        .range([size.value.height - margin.bottom, margin.top])

    // 2. Render Axes
    svg.append('g')
        .attr('transform', `translate(0, ${size.value.height - margin.bottom})`)
        .call(d3.axisBottom(xScale).ticks(10))
        .append('text')
        .attr('x', size.value.width - margin.right)
        .attr('y', -10)
        .attr('fill', 'black')
        .attr('text-anchor', 'end')
        .text('Page Count')

    svg.append('g')
        .attr('transform', `translate(${margin.left}, 0)`)
        .call(d3.axisLeft(yScale))
        .append('text')
        .attr('transform', 'rotate(-90)')
        .attr('y', 15)
        .attr('x', -margin.top)
        .attr('fill', 'black')
        .attr('text-anchor', 'end')
        .text('Avg Rating')

    // 3. Draw Points
    svg.append('g')
        .selectAll('circle')
        .data(books.value)
        .join('circle')
        .attr('cx', d => xScale(d.pageCount))
        .attr('cy', d => yScale(d.rating_average))
        .attr('r', 5)
        .attr('fill', '#1867C0')
        .attr('opacity', 0.6)
        .append('title') // Simple hover tooltip
        .text(d => `${d.title}\nRating: ${d.rating_average}\nPages: ${d.pageCount}`)
}

watch([books, size], ([bVal, sVal]) => {
    if (!isEmpty(bVal) && sVal.width > 0 && sVal.height > 0) {
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
    <div class="chart-wrapper" ref="chartContainer">
        <svg id="scatter-svg" width="100%" height="100%"></svg>
    </div>
</template>

<style scoped>
.chart-wrapper {
    width: 100%;
    height: 100%;
    min-height: 450px;
}
</style>