<script setup lang="ts">
import { isEmpty} from 'lodash'
import { ref, watch, onMounted} from 'vue'
import * as d3 from 'd3'

interface Book {
    genre: string;
    pageCount: number;
}

const books = ref<Book[]>([])
const size = ref({ width: 0, height: 0 })
const margin = { left: 20, right: 250, top: 40, bottom: 40 }
const chartContainer = ref<HTMLElement | null>(null)

async function read() {
    try {
        const dataPath = new URL('../../data/top_1000_most_swapped_books.csv', import.meta.url).href;
        const csvData = await d3.csv(dataPath, (d) => {
            return {
                genre: d.genre as string,
                pageCount: +d.pageCount!
            }
        })
        // Filter for pages between 300 and 350
        books.value = csvData.filter(d => d.pageCount >= 300 && d.pageCount <= 350)
    } catch (e) {
        console.error("Error loading CSV:", e)
    }
}

// Resize graph depending on browser window
function onResize() {
    if (!chartContainer.value) return
    size.value = { 
        width: chartContainer.value.clientWidth, 
        height: chartContainer.value.clientHeight 
    }
}

function initChart() {
    const svg = d3.select('#pie-svg')
    svg.selectAll('*').remove()

    const genreCounts = d3.rollup(books.value, v => v.length, d => d.genre)
    const pieData = Array.from(genreCounts, ([name, value]) => ({ name, value }))
        .sort((a, b) => b.value - a.value)

    const width = size.value.width
    const height = size.value.height
    const radius = Math.min(width - margin.right, height) / 2 - margin.top

    // Setup Red scale
    const maxCount = d3.max(pieData, d => d.value) || 0
    const minCount = d3.min(pieData, d => d.value) || 0
    
    const colorScale = d3.scaleSequential()
        .domain([minCount, maxCount]) // Light red for min, Dark red for max
        .interpolator(d3.interpolateReds)

    // Generate pie chart
    const pie = d3.pie<{name: string, value: number}>()
        .value(d => d.value)
        .sort(null)

    const arc = d3.arc<d3.PieArcDatum<{name: string, value: number}>>()
        .innerRadius(0) 
        .outerRadius(radius)

    const mainGroup = svg.append('g')
        .attr('transform', `translate(${(width - margin.right) / 2 + margin.left}, ${height / 2})`)

    mainGroup.selectAll('path')
        .data(pie(pieData))
        .join('path')
        .attr('d', arc)
        .attr('fill', d => colorScale(d.data.value))
        .attr('stroke', '#ccc')
        .style('stroke-width', '1px')

    // Generate legend
    const legend = svg.append('g')
        .attr('transform', `translate(${width - margin.right + 40}, ${height / 2 - (10 * 11) - 110})`);

    pieData.slice(0, 20).forEach((d, i) => {
        const row = legend.append('g')
            .attr('transform', `translate(0, ${i * 22})`);

        row.append('rect')
            .attr('width', 14)
            .attr('height', 14)
            .attr('fill', colorScale(d.value))
            .attr('stroke', '#ccc');

        row.append('text')
            .attr('x', 24)
            .attr('y', 12)
            .style('font-size', '13px')
            .style('white-space', 'nowrap') 
            .text(`${d.name} (${d.value})`);
    });
}

watch([books, size], ([bVal, sVal]) => {
    if (!isEmpty(bVal) && sVal.width > 0 && sVal.height > 0) {
        initChart()
    }
}, { deep: true })

onMounted(async () => {
    read()
    onResize()
})

</script>

<template>
    <div class="chart-wrapper" ref="chartContainer">
        <svg id="pie-svg" width="100%" height="100%"></svg>
    </div>
</template>

<style scoped>
.chart-wrapper {
    width: 100%;
    height: 100%;
}
</style>