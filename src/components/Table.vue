<template>
    <div class="container">
        <h2 class="heading">Satellite Objects</h2>

        <table class="data-table">
            <thead>
                <tr>
                    <th>Name</th>
                    <th>NORAD Cat ID</th>
                    <th>Orbit Code</th>
                    <th>Object Type</th>
                    <th>Country</th>
                    <th>Launch Date</th>
                </tr>
            </thead>
            <tbody>
                <!-- Show skeleton rows if loading -->
                <template v-if="loading">
                    <tr v-for="n in 10" :key="n">
                        <td v-for="m in 6" :key="m">
                            <div class="skeleton"></div>
                        </td>
                    </tr>
                </template>

                <!-- Show data when loaded -->
                <template v-else>
                    <tr v-for="item in paginatedData" :key="item.noradCatId">
                        <td>{{ item.name || '-' }}</td>
                        <td>{{ item.noradCatId || '-' }}</td>
                        <td>{{ item.orbitCode || '-' }}</td>
                        <td>{{ item.objectType || '-' }}</td>
                        <td>{{ item.countryCode || '-' }}</td>
                        <td>{{ item.launchDate || '-' }}</td>
                    </tr>
                </template>
            </tbody>
        </table>

        <!-- Pagination -->
        <div class="pagination" v-if="!loading">
            <button @click="prevPage" :disabled="currentPage === 1">Prev</button>
            <span>Page {{ currentPage }} / {{ totalPages }}</span>
            <button @click="nextPage" :disabled="currentPage === totalPages">Next</button>

            <label>
                Rows:
                <select v-model.number="pageSize">
                    <option v-for="size in pageSizes" :key="size" :value="size">
                        {{ size }}
                    </option>
                </select>
            </label>
        </div>
    </div>
</template>
  
<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue'
import axios from 'axios'

const attributes = [
    'name',
    'noradCatId',
    'orbitCode',
    'objectType',
    'countryCode',
    'launchDate'
]

const data = ref<any[]>([])
const currentPage = ref(1)
const pageSize = ref(10)
const pageSizes = [10, 20, 30, 50, 100]
const loading = ref(true)

// API fetch function
const fetchData = async () => {
    try {
        const queryString = attributes.join(',')
        const response = await axios.get(
            `https://backend.digantara.dev/v1/satellites?attributes=${queryString}`
        )
        data.value = response.data.data
    } catch (error) {
        console.error('Error fetching data:', error)
    } finally {
        loading.value = false
    }
}

onMounted(() => {
    fetchData()
})

const totalPages = computed(() =>
    Math.ceil(data.value.length / pageSize.value)
)

const paginatedData = computed(() => {
    const start = (currentPage.value - 1) * pageSize.value
    const end = start + pageSize.value
    return data.value.slice(start, end)
})

const nextPage = () => {
    if (currentPage.value < totalPages.value) currentPage.value++
}

const prevPage = () => {
    if (currentPage.value > 1) currentPage.value--
}

watch(pageSize, () => {
    currentPage.value = 1
})
</script>
  
<style scoped>
.container {
    background-color: #10131c;
    padding: 24px;
    border-radius: 10px;
    color: #dcdcdc;
    font-family: 'Segoe UI', sans-serif;
    max-width: 1000px;
    margin: 0 auto;
}

.heading {
    font-size: 26px;
    font-weight: 600;
    margin-bottom: 20px;
    color: #f0f0f0;
}

.data-table {
    width: 100%;
    border-collapse: collapse;
    border-radius: 8px;
    overflow: hidden;
}

.data-table th,
.data-table td {
    padding: 12px;
    text-align: center;
    border-bottom: 1px solid #2a2d3c;
}

.data-table th {
    background-color: #181b25;
    color: #c4c4c4;
    font-weight: 600;
    font-size: 14px;
}

.data-table tr:nth-child(even) {
    background-color: #161920;
}

.data-table tr:nth-child(odd) {
    background-color: #1b1e27;
}

.data-table tr:hover {
    background-color: #212433;
    color: #ffffff;
    transition: 0.3s ease;
}

/* Skeleton loader */
.skeleton {
    height: 14px;
    width: 80%;
    margin: 0 auto;
    background: linear-gradient(90deg,
            #2a2d3c 25%,
            #3a3f4d 50%,
            #2a2d3c 75%);
    background-size: 200% 100%;
    animation: loading 1.2s infinite ease-in-out;
    border-radius: 4px;
}

@keyframes loading {
    0% {
        background-position: 200% 0;
    }

    100% {
        background-position: -200% 0;
    }
}

.pagination {
    display: flex;
    justify-content: flex-end;
    align-items: center;
    margin-top: 20px;
    gap: 12px;
    flex-wrap: wrap;
}

.pagination button {
    background-color: #242a38;
    border: 1px solid #3a3f4d;
    color: #cfcfcf;
    padding: 8px 14px;
    font-size: 13px;
    border-radius: 6px;
    cursor: pointer;
    transition: all 0.2s ease;
}

.pagination button:hover:not(:disabled) {
    background-color: #33394d;
    color: #fff;
}

.pagination button:disabled {
    background-color: #2a2d3c;
    color: #777;
    cursor: not-allowed;
}

.pagination select {
    background-color: #181b25;
    color: #ddd;
    padding: 7px 10px;
    border: 1px solid #3a3f4d;
    border-radius: 4px;
    outline: none;
}

.pagination label {
    font-size: 13px;
    color: #cfcfcf;
}
</style>
  