<script setup lang="ts">
import { isEmpty} from 'lodash'
import { ref, watch, onMounted} from 'vue'
import * as d3 from 'd3'

const data = ref<number[]>([])
const size = ref({ width: 0, height: 0 })
const margin = { left: 60, right: 30, top: 40, bottom: 60 }
const chartContainer = ref<HTMLElement | null>(null)

async function read() {
    try {
        const readFromCSV = await d3.csv('../../data/top_1000_most_swapped_books.csv', (d: d3.DSVRowString<'pageCount'>) => {
            return { 
                pageCount: +d.pageCount! 
            } as { pageCount: number }
        })
        
        data.value = readFromCSV
            .map(d => d.pageCount)
            .filter(val => !isNaN(val))
            
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
    const svg = d3.select('#histogram-svg')
    svg.selectAll('*').remove()

    const x = d3.scaleLinear()
        .domain([0, d3.max(data.value) || 1000])
        .range([margin.left, size.value.width - margin.right])
        .nice()

    const histogram = d3.bin()
        .domain(x.domain() as [number, number])
        .thresholds(x.ticks(30))

    const bins = histogram(data.value)

    const y = d3.scaleLinear()
        .domain([0, d3.max(bins, d => d.length) || 0])
        .range([size.value.height - margin.bottom, margin.top])
        .nice()

    svg.append('g')
        .attr('transform', `translate(0, ${size.value.height - margin.bottom})`)
        .call(d3.axisBottom(x).ticks(20))
        .append('text')
        .attr('x', size.value.width / 2)
        .attr('y', 40)
        .attr('fill', 'black')
        .text('Page Count')
        .style('font-size', '18px')
        .style('font-weight', 'bold')

    svg.append('g')
        .attr('transform', `translate(${margin.left}, 0)`)
        .call(d3.axisLeft(y))
        .append('text')
        .attr('transform', 'rotate(-90)')
        .attr('y', -45)
        .attr('x', -size.value.height / 2)
        .attr('fill', 'black')
        .text('Frequency')
        .style('font-size', '18px')
        .style('font-weight', 'bold')

    // Highlight bin with highest frequency
    const maxVal = d3.max(bins, d => d.length); 

    svg.append('g')
        .selectAll('rect')
        .data(bins)
        .join('rect')
        .attr('x', d => x(d.x0!) + 1)
        .attr('width', d => Math.max(0, x(d.x1!) - x(d.x0!) - 1))
        .attr('y', d => y(d.length))
        .attr('height', d => y(0) - y(d.length))
        // Color highest bar
        .attr('fill', d => d.length === maxVal ? '#FF5252' : '#4CAF50');

    // Add label to highest bar
    svg.append('g')
        .selectAll('text')
        .data(bins.filter(d => d.length === maxVal))
        .join('text')
        .attr('x', d => x(d.x0!) + (x(d.x1!) - x(d.x0!)) / 2)
        .attr('y', d => y(d.length) - 10)
        .attr('text-anchor', 'middle')
        .style('font-weight', 'bold')
        .style('font-size', '16px')
        .text(d => `${Math.round(d.x0!)} - ${Math.round(d.x1!)} pages`);
}

watch([data, size], ([dVal, sVal]) => {
    if (!isEmpty(dVal) && sVal.width > 0 && sVal.height > 0) {
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
        <svg id="histogram-svg" width="100%" height="100%"></svg>
    </div>
</template>

<style scoped>
.chart-wrapper {
    width: 100%;
    height: 100%;
    min-height: 400px;
}
</style>