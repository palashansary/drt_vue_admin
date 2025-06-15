<template>
    <div class="container">
        <h2 class="heading">Satellite Objects</h2>

        <!-- Filter Section -->
        <div class="filters">
            <input v-model="searchName" @keyup.enter="applyFilters" placeholder="Search Name" />
            <input v-model="searchNoradId" @keyup.enter="applyFilters" placeholder="Search NORAD ID" />

            <div class="filter-section">
                <h3>Object Type</h3>
                <div class="filter-options">
                    <button v-for="type in objectTypes" :key="type" :class="{ active: selectedObjectTypes.includes(type) }"
                        @click="toggleObjectType(type)">
                        {{ type }} ({{ getCountByType(type) }})
                    </button>
                </div>
            </div>

            <div class="filter-section">
                <h3>Orbit Code</h3>
                <div class="filter-options">
                    <button v-for="orbit in orbitCodes" :key="orbit" :class="{ active: selectedOrbitCodes.includes(orbit) }"
                        @click="toggleOrbitCode(orbit)">
                        {{ orbit }}
                    </button>
                </div>
            </div>

            <div class="filter-actions">
                <button @click="applyFilters" class="apply">Apply</button>
                <button @click="clearAllFilters" class="clear">Clear</button>
            </div>
        </div>

        <div class="result-info">
            Showing {{ filteredData.length }} object<span v-if="filteredData.length !== 1">s</span>
        </div>

        <!-- Table -->
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
                <template v-if="loading">
                    <tr v-for="n in 10" :key="n">
                        <td colspan="6">
                            <div class="skeleton"></div>
                        </td>
                    </tr>
                </template>
                <template v-else>
                    <tr v-for="item in paginatedData" :key="item.noradCatId">
                        <td>{{ item.name || '-' }}</td>
                        <td>{{ item.noradCatId || '-' }}</td>
                        <td>{{ item.orbitCode || '-' }}</td>
                        <td>{{ item.objectType || '-' }}</td>
                        <td>{{ item.countryCode || '-' }}</td>
                        <td>{{ item.launchDate || '-' }}</td>
                    </tr>
                    <tr v-if="paginatedData.length === 0">
                        <td colspan="6">No results found</td>
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
                    <option v-for="size in pageSizes" :key="size" :value="size">{{ size }}</option>
                </select>
            </label>
        </div>
    </div>
</template>
  
<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import axios from 'axios'

const data = ref([])
const filteredData = ref([])
const currentPage = ref(1)
const pageSize = ref(10)
const pageSizes = [10, 20, 30, 50, 100]
const loading = ref(true)

const searchName = ref('')
const searchNoradId = ref('')
const selectedObjectTypes = ref([])
const selectedOrbitCodes = ref([])

const objectTypes = ['All Objects', 'PAYLOAD', 'DEBRIS', 'ROCKET BODY']
const orbitCodes = ['LEO', 'LEO1', 'LEO2', 'LEO3', 'LEO4', 'MEO']

const fetchData = async () => {
    try {
        const response = await axios.get(
            'https://backend.digantara.dev/v1/satellites?attributes=name,noradCatId,orbitCode,objectType,countryCode,launchDate'
        )
        data.value = response.data.data
        filteredData.value = data.value
    } catch (error) {
        console.error('Error fetching data:', error)
    } finally {
        loading.value = false
    }
}

onMounted(fetchData)

const getCountByType = (type) => {
    if (type === 'All Objects') return data.value.length
    return data.value.filter((item) => item.objectType === type).length
}

const applyFilters = () => {
    loading.value = true
    setTimeout(() => {
        let result = data.value

        if (searchName.value.trim())
            result = result.filter((item) =>
                item.name?.toLowerCase().includes(searchName.value.toLowerCase())
            )

        if (searchNoradId.value.trim())
            result = result.filter((item) =>
                String(item.noradCatId).includes(searchNoradId.value)
            )

        if (selectedObjectTypes.value.length && !selectedObjectTypes.value.includes('All Objects'))
            result = result.filter((item) =>
                selectedObjectTypes.value.includes(item.objectType)
            )

        if (selectedOrbitCodes.value.length)
            result = result.filter((item) =>
                selectedOrbitCodes.value.includes(item.orbitCode?.replace(/[{}]/g, ''))
            )


        filteredData.value = result
        currentPage.value = 1
        loading.value = false
    }, 500)
}

const clearAllFilters = () => {
    searchName.value = ''
    searchNoradId.value = ''
    selectedObjectTypes.value = []
    selectedOrbitCodes.value = []
    applyFilters()
}

const toggleObjectType = (type) => {
    if (type === 'All Objects') {
        selectedObjectTypes.value = ['All Objects']
    } else {
        if (selectedObjectTypes.value.includes('All Objects')) {
            selectedObjectTypes.value = []
        }
        const index = selectedObjectTypes.value.indexOf(type)
        if (index > -1) selectedObjectTypes.value.splice(index, 1)
        else selectedObjectTypes.value.push(type)
    }
}

const toggleOrbitCode = (code) => {
    const index = selectedOrbitCodes.value.indexOf(code)
    if (index > -1) selectedOrbitCodes.value.splice(index, 1)
    else selectedOrbitCodes.value.push(code)
}

const totalPages = computed(() =>
    Math.ceil(filteredData.value.length / pageSize.value)
)

const paginatedData = computed(() => {
    const start = (currentPage.value - 1) * pageSize.value
    const end = start + pageSize.value
    return filteredData.value.slice(start, end)
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
    max-width: 1200px;
    margin: 0 auto;
}

.heading {
    font-size: 26px;
    font-weight: 600;
    margin-bottom: 20px;
    color: #f0f0f0;
}

.filters {
    display: flex;
    flex-wrap: wrap;
    gap: 14px;
    margin-bottom: 20px;
}

.filters input {
    padding: 6px 12px;
    height: 30px;
    border: 1px solid #333;
    background: #181b25;
    color: #eee;
    border-radius: 6px;
}

.filter-section {
    display: flex;
    flex-direction: column;
}

.filter-options {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
    margin-top: 8px;
}

.filter-options button {
    background: #242a38;
    border: 1px solid #3a3f4d;
    padding: 10px 14px;
    border-radius: 6px;
    color: #ccc;
    cursor: pointer;
}

.filter-options button.active {
    background: #334;
    border-color: #556;
    color: #fff;
}

.filter-actions {
    display: flex;
    gap: 8px;
    align-items: flex-end;
}

.filter-actions button {
    padding: 10px 14px;
    border-radius: 6px;
    border: none;
    background: green;
    color: #ddd;
    cursor: pointer;
}

.filter-actions .clear {
    padding: 10px 14px;
    border-radius: 6px;
    border: none;
    background: red;
    color: #ddd;
    cursor: pointer;
}

.result-info {
    margin-bottom: 10px;
    font-size: 14px;
    color: #aaa;
}

.data-table {
    width: 100%;
    border-collapse: collapse;
}

.data-table th,
.data-table td {
    padding: 12px;
    text-align: center;
    border-bottom: 1px solid #2a2d3c;
}

.data-table th {
    background: #181b25;
    color: #ccc;
}

.data-table tr:nth-child(even) {
    background: #161920;
}

.data-table tr:nth-child(odd) {
    background: #1b1e27;
}

.data-table tr:hover {
    background: #212433;
}

.skeleton {
    height: 20px;
    background: linear-gradient(90deg, #2a2d3c 25%, #3a3f4d 50%, #2a2d3c 75%);
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
    gap: 12px;
    margin-top: 20px;
}

.pagination button,
.pagination select {
    padding: 8px 12px;
    border: none;
    background: #242a38;
    color: #ccc;
    border-radius: 4px;
}
</style>
  