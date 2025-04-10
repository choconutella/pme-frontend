<script setup lang="ts">
    import { ref, onMounted, reactive, computed } from 'vue'
    import Search from '@/components/atoms/Search.vue'

    // Define the types to match your API response
    interface Analyzer {
        code: string;
        name: string;
    }

    interface InternalStats {
        noResult: number
        mean: number
        sd: number
        cv: number
    }

    interface ExternalStats {
        mean: number
        meanGroup: number
        bias: number
    }

    interface QcDetail {
        testCd: string
        testNm: string
        internal: InternalStats
        external: ExternalStats
        tea: number
        sigma: number
    }

    interface QcData {
        id: number
        anzId: string
        lotNo: string
        startDate: string
        endDate: string
        details: QcDetail[]
    }

    interface ApiResponse {
        ok: boolean
        data: QcData
        error: any
    }

    // Set up reactive state
    const qcData = ref<QcDetail[]>([])
    const analyzers = ref<Analyzer[]>([])
    const isLoading = ref(false)
    const isLoadingAnalyzers = ref(true)
    const error = ref<string | null>(null)
    const analyzerError = ref<string | null>(null)

    // Form data for filtering
    const filters = reactive({
        analyzer: '',  
        lotNo: '',
        startDate: '',
        endDate: ''
    })

    // Function to fetch analyzers
    const fetchAnalyzers = () => {
        isLoadingAnalyzers.value = true
        analyzerError.value = null
        
        fetch(`${import.meta.env.VITE_API_URL}/analyzer`)
            .then(response => {
                if (!response.ok) {
                    throw new Error(`HTTP error! Status: ${response.status}`)
                }
                return response.json()
            })
            .then((responseData: { ok: boolean, data: Analyzer[], error: any }) => {
                if (responseData.ok && responseData.data) {
                    analyzers.value = responseData.data
                    
                    // Set default analyzer if available
                    if (analyzers.value.length > 0) {
                        filters.analyzer = analyzers.value[0].code
                    }
                } else {
                    throw new Error(responseData.error?.message || 'Unknown error')
                }
            })
            .catch(err => {
                console.error('Error fetching analyzers:', err)
                analyzerError.value = err.message
                analyzers.value = []
            })
            .finally(() => {
                isLoadingAnalyzers.value = false
            })
    }

    // Add a function to format date for the API
    const formatDateForApi = (dateString: string): string => {
        // If empty, return empty string
        if (!dateString) return '';
        
        // Convert input date (YYYY-MM-DD) to required format (YYYYMMDD)
        return dateString.replace(/-/g, '');
    }

    // Function to fetch data
    const fetchQcData = () => {

        isLoading.value = true;
        error.value = null;
        
        // Format dates for API
        const formattedStartDate = formatDateForApi(filters.startDate);
        const formattedEndDate = formatDateForApi(filters.endDate);
        
        const apiUrl = `${import.meta.env.VITE_API_URL}/qc/calc?anz=${filters.analyzer}&start=${formattedStartDate}&end=${formattedEndDate}&lotno=${filters.lotNo}`;
        
        console.log("Fetching QC data from:", apiUrl);
        
        fetch(apiUrl)
            .then(response => {
                if (!response.ok) {
                    throw new Error(`HTTP error! Status: ${response.status}`);
                }
                return response.json();
            })
            .then((responseData: ApiResponse) => {
                if (responseData.ok && responseData.data) {
                    qcData.value = responseData.data.details;
                    console.log('QC data loaded:', qcData.value);
                } else {
                    throw new Error(responseData.error?.message || 'Unknown error');
                }
            })
            .catch(err => {
                console.error('Error fetching QC data:', err);
                error.value = err.message;
                qcData.value = []; // Clear previous data on error
            })
            .finally(() => {
                isLoading.value = false;
            });
    }

    // Format number to 2 decimal places
    const formatNumber = (num: number): string => {
        return num === 0 ? '0.00' : num.toFixed(2)
    }
    
    // Computed property to check if form is valid
    const isFormValid = computed(() => {
        return !!filters.analyzer && !!filters.startDate && !!filters.endDate;
    })
    
    // Initialize data on component mount
    onMounted(() => {
        fetchAnalyzers();
        // Don't fetch QC data automatically
    })

    // Function to reset filters
    const resetFilters = () => {
        // Reset filters to default values
        filters.lotNo = ''
        filters.startDate = ''
        filters.endDate = ''
        
        // Keep the analyzer selection if available
        if (analyzers.value.length > 0) {
            filters.analyzer = analyzers.value[0].code
        } else {
            filters.analyzer = ''
        }
        
        // Clear QC data as well
        qcData.value = []
        error.value = null
    }
</script>

<template>
    <div>
        <div class="my-5">
            <div class="text-center">
                <h1 class="text-2xl font-semibold">QC Performance</h1>
            </div>
        </div>
        <div class="w-full flex flex-wrap py-2">
            <div class="w-2/12 px-3">
                <label for="analyzer-select" class="block text-sm font-medium text-gray-700">Analyzer</label>
                <select 
                    id="analyzer-select" 
                    v-model="filters.analyzer" 
                    class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500"
                    :disabled="isLoadingAnalyzers"
                >
                    <option v-if="isLoadingAnalyzers" value="">Loading...</option>
                    <option v-else-if="analyzers.length === 0" value="">No analyzers available</option>
                    <option v-else value="" disabled selected>Select an analyzer</option>
                    <option v-for="analyzer in analyzers" :key="analyzer.code" :value="analyzer.code">
                        {{ analyzer.name }}
                    </option>
                </select>
                <div v-if="analyzerError" class="mt-1 text-xs text-red-500">
                    {{ analyzerError }}
                </div>
            </div>
            <div class="w-2/12 px-3">
                <label for="lotno-input" class="block text-sm font-medium text-gray-700">Lot No</label>
                <input 
                    id="lotno-input" 
                    type="text" 
                    v-model="filters.lotNo" 
                    placeholder="e.g. 20201102" 
                    class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500"
                />
            </div>
            <div class="w-2/12 px-3">
                <label for="start-date-input" class="block text-sm font-medium text-gray-700">Start Date</label>
                <input 
                    id="start-date-input" 
                    type="date" 
                    v-model="filters.startDate" 
                    class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500"
                />
            </div>
            <div class="w-2/12 px-3">
                <label for="end-date-input" class="block text-sm font-medium text-gray-700">End Date</label>
                <input 
                    id="end-date-input" 
                    type="date" 
                    v-model="filters.endDate" 
                    class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500"
                />
            </div>
            <div class="px-2">
                <button 
                    @click="fetchQcData" 
                    class="mt-4 bg-blue-500 hover:bg-blue-700 text-white py-2 px-4 rounded"
                    :disabled="isLoading || !isFormValid"
                    :class="{ 'opacity-50 cursor-not-allowed': isLoading || !isFormValid }"
                >
                    <span v-if="isLoading">Loading...</span>
                    <span v-else>Generate</span>
                </button>
            </div>
            <div class="px-2">
                <button 
                    @click="resetFilters" 
                    class="mt-4 bg-gray-300 hover:bg-gray-400 text-gray-800 py-2 px-4 rounded"
                    :disabled="isLoading"
                    :class="{ 'opacity-50 cursor-not-allowed': isLoading }"
                >
                    Reset
                </button>
            </div>
        </div>

        <!-- Loading state -->
        <div v-if="isLoading" class="text-center py-10">
            <div class="inline-block animate-spin rounded-full h-8 w-8 border-t-2 border-b-2 border-blue-500"></div>
            <p class="mt-2 text-gray-600">Loading QC data...</p>
        </div>
        
        <!-- Empty state, show when no filter values are provided -->
        <div v-else-if="!filters.analyzer || !filters.startDate || !filters.endDate" class="text-center py-10">
            <Search  />
            <p class="mt-3 text-gray-600">Please select analyzer, lot no, date range and click Generate</p>
        </div>

        <!-- Error state -->
        <div v-else-if="error" class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded my-4">
            <p>Error loading QC data: {{ error }}</p>
            <button 
                @click="fetchQcData" 
                class="mt-2 bg-red-500 hover:bg-red-700 text-white py-1 px-3 rounded text-sm"
            >
                Try Again
            </button>
        </div>
        
        <!-- Data table -->
        <div v-else>
            <table class="w-full border">
                <thead>
                    <tr>
                        <th rowspan="2" class="w-2/12">Test Name</th>
                        <th colspan="4">Calculated</th>
                        <th colspan="3">External</th>
                        <th rowspan="2" class="w-1/12">TEA</th>
                        <th rowspan="2" class="w-1/12">Sigma</th>
                    </tr>
                    <tr>
                        <th class="w-1/12">No. of Result</th>
                        <th class="w-1/12">Mean</th>
                        <th class="w-1/12">SD</th>
                        <th class="w-1/12">CV</th>
                        <th class="w-1/12">Mean</th>
                        <th class="w-1/12">Mean Group</th>
                        <th class="w-1/12">Bias</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-if="qcData.length === 0">
                        <td colspan="10" class="border py-4 text-center text-gray-500">
                            No QC data available
                        </td>
                    </tr>
                    <tr v-for="(detail, index) in qcData" :key="detail.testCd">
                        <td class="border py-1.5 px-2">{{ detail.testNm }}</td>
                        <td class="border py-1.5 text-center">{{ detail.internal.noResult }}</td>
                        <td class="border py-1.5 text-center">{{ formatNumber(detail.internal.mean) }}</td>
                        <td class="border py-1.5 text-center">{{ formatNumber(detail.internal.sd) }}</td>
                        <td class="border py-1.5 text-center">{{ formatNumber(detail.internal.cv) }}</td>
                        <td class="border py-1.5 text-center">{{ formatNumber(detail.external.mean) }}</td>
                        <td class="border py-1.5 text-center">{{ formatNumber(detail.external.meanGroup) }}</td>
                        <td class="border py-1.5 text-center">{{ formatNumber(detail.external.bias) }}</td>
                        <td class="border py-1.5 text-center">{{ formatNumber(detail.tea) }}</td>
                        <td class="border py-1.5 text-center">{{ formatNumber(detail.sigma) }}</td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<style scoped>
    th {
        border: 2px solid #ccc;
        padding: 8px 4px;
        background-color: #f0f0f0;
    }
    
    .border {
        border: 1px solid #ccc;
    }
    
    /* Alternating row colors for better readability */
    tbody tr:nth-child(even) {
        background-color: #f7f7f7;
    }
    
    tbody tr:hover {
        background-color: #f0f8ff;
    }
</style>