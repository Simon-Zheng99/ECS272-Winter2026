<script setup lang="ts">
import { ref, onMounted } from 'vue';
import * as d3 from 'd3';

interface BookData {
    pageCount: number;
    rating_average: number;
    year: number;
    language: string;
}

const chartContainer = ref<HTMLElement | null>(null);
const books = ref<BookData[]>([]);
const margin = { left: 70, right: 70, top: 50, bottom: 50 };
const width = 800 - margin.left - margin.right;
const height = 300 - margin.top - margin.bottom;
//Manually set colors, Green aligns with histogram
const myColors = [
    "#4CAF50", 
    "#2E86AB",
    "#A23B72", 
    "#F18F01", 
    "#C73E1D", 
    "#3D5A80", 
    "#7678ED", 
    "#136F63", 
    "#8338EC",
    "#588157",
    "#E9C46A",
    "#E76F51",
    "#264653" 
];
const colorScale = d3.scaleOrdinal()
    .domain([...new Set(books.value.map(d => d.language))])
    .range(myColors);

async function read() {
    try {
        const dataPath = new URL('../../data/top_1000_most_swapped_books.csv', import.meta.url).href; 
        const data = await d3.csv(dataPath, (d) => {
            return {
                pageCount: +d.pageCount, 
                rating_average: +d.rating_average,
                year: +d.publicationYear, 
                language: d.language
            }
        });
        books.value = data as BookData[];
        
        if (books.value.length > 0) {
            drawChart();
        }
    } catch (e) {
        console.error("Data loading failed:", e);
    }
}

const axisLabels: Record<string, string> = {
    "rating_average": "Avg. Rating",
    "pageCount": "Total Pages",
    "year": "Published Year",
    "language": "Language"
};

function drawChart() {
    if (!chartContainer.value) return;
    const container = d3.select(chartContainer.value);

    const svg = container.append("svg")
        .attr("viewBox", `0 0 ${width + margin.left + margin.right} ${height + margin.top + margin.bottom}`)
        .append("g")
        .attr("transform", `translate(${margin.left},${margin.top})`);

    // Dimensions
    const dimensions = ["rating_average", "pageCount", "year", "language"] as const;

    // Scales
    const yScales: any = {};
    dimensions.forEach(dim => {
        if (dim === 'language') {
            yScales[dim] = d3.scalePoint()
                .domain([...new Set(books.value.map(d => d.language))].sort())
                .range([height, 0]);
        } else {
            yScales[dim] = d3.scaleLinear()
                .domain(d3.extent(books.value, d => d[dim] as number) as [number, number])
                .range([height, 0]);
        }
    });

    const xScale = d3.scalePoint()
        .range([0, width])
        .domain(dimensions);

    // Draw Parallel Coordinate Lines
    const lineGenerator = d3.line();
    svg.append("g")
        .selectAll("path")
        .data(books.value)
        .enter()
        .append("path")
        .attr("d", (d) => {
            const points: [number, number][] = dimensions.map(p => [
                xScale(p)!, 
                yScales[p](d[p])
            ]);
            return lineGenerator(points);
        })
        .style("fill", "none")
        .style("stroke", d => colorScale(d.language) as string)
        .style("opacity", 0.15)
        .style("stroke-width", "1px")

    // Draw Axes
    svg.selectAll(".axis")
        .data(dimensions).enter()
        .append("g")
        .attr("transform", d => `translate(${xScale(d)})`)
        .each(function(d) { d3.select(this).call(d3.axisLeft(yScales[d])); })
        .append("text")
        .attr("y", -20)
        .text(d => axisLabels[d] || d)
        .style("text-anchor", "middle")
        .style("fill", "#2c3e50")
        .style("font-size", "12px")
        .style("font-weight", "bold");
}

onMounted(() => {
    read();
});
</script>

<template>
  <div ref="chartContainer" style="width: 100%; height: 500px;"></div>
</template>