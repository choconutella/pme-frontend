<script setup lang="ts">
    import BackButton from '@/components/molecules/BackButton.vue'
    import AddTestButton from '@/components/molecules/AddTestButton.vue'
    import TableInput from '@/components/molecules/TableInput.vue'

    import { ref, onMounted, computed } from 'vue'
    
    // Define props for edit mode
    const props = defineProps<{
        qcId?: string
    }>()

    // Create reactive state for analyzers with proper type
    interface SelectType {
        value: string
        name: string
    }

    // Define the type for table rows
    interface TableRow {
        testName: string | null
        mean: number | null
        meanGroup: number | null
    }

    interface FormData {
        anzId: string
        name: string
        periode: string
        activityDate: Date
    }
    
    const analyzers = ref<SelectType[]>([])
    const pmes = ref<SelectType[]>([])

    // Status messages
    const statusMessage = ref('')
    const isSuccess = ref(false)
    const isLoading = ref(false)
    const isLoadingAnalyzers = ref(false)
    const isLoadingPmes = ref(false)

    // Form data state
    const formData = ref<FormData>(
        {
            anzId: '',
            name: '',
            periode: '',
            activityDate: new Date()
        }
    )

    // Create reactive state for table rows
    const tableRows = ref<TableRow[]>([])

    // Determine if we are in edit mode
    const isEditMode = computed(() => !!props.qcId)

    // Form title changes based on mode
    const formTitle = computed(() => isEditMode.value ? 'Edit QC PME' : 'New QC PME')

    const isFormValid = computed(() => {
        // Check if basic form fields are filled
        const hasRequiredFields = 
            !!formData.value.anzId && 
            !!formData.value.name && 
            !!formData.value.periode && 
            !!formData.value.activityDate
        
        // Check if we have at least one table row
        const hasTableRows = tableRows.value.length > 0
        
        // Check if all table rows have valid data
        const hasValidTableData = tableRows.value.every(row => {
            return (
                // Test name is required
                !!row.testName && 
                // Mean and meanGroup should be valid numbers if provided
                (row.mean === null || (typeof row.mean === 'number' && !isNaN(row.mean))) &&
                (row.meanGroup === null || (typeof row.meanGroup === 'number' && !isNaN(row.meanGroup)))
            )
        })
        
        return hasRequiredFields && hasTableRows && hasValidTableData
    })

    // Function to fetch analyzers from API
    const fetchAnalyzers = () => {
        isLoadingAnalyzers.value = true
        
        fetch(`${import.meta.env.VITE_API_URL}/analyzer`)
            .then(response => {
                if (!response.ok) {
                    throw new Error(`HTTP error! Status: ${response.status}`)
                }
                return response.json()
            })
            .then(response => {
                if (response.ok && response.data && Array.isArray(response.data)) {
                    // Map API data to the format expected by the select component
                    analyzers.value = response.data.map((analyzer: { code: string; name: string }) => ({
                        value: analyzer.code,
                        name: analyzer.name
                    }))
                } else {
                    console.error('Invalid analyzer response format:', response)
                    analyzers.value = []
                }
            })
            .catch(error => {
                console.error('Error fetching analyzers:', error)
                analyzers.value = []
            })
            .finally(() => {
                isLoadingAnalyzers.value = false
            })
    }

    // Function to fetch PMEs from API
    const fetchPmes = () => {
        isLoadingPmes.value = true
        
        fetch(`${import.meta.env.VITE_API_URL}/pme`)
            .then(response => {
                if (!response.ok) {
                    throw new Error(`HTTP error! Status: ${response.status}`)
                }
                return response.json()
            })
            .then(response => {
                if (response.ok && response.data && Array.isArray(response.data)) {
                    // Map API data to the format expected by the select component
                    pmes.value = response.data.map((pme: { code: string; name: string }) => ({
                        value: pme.code,
                        name: pme.name
                    }))
                } else {
                    console.error('Invalid PME response format:', response)
                    pmes.value = []
                }
            })
            .catch(error => {
                console.error('Error fetching PMEs:', error)
                pmes.value = []
            })
            .finally(() => {
                isLoadingPmes.value = false
            })
    }
    
    // Fetch analyzers and PMEs on component mount
    onMounted(() => {
        fetchAnalyzers()
        fetchPmes()
        
        if (isEditMode.value) {
            loadQcData()
        }
    })

    // Counter to generate unique test names
    let testCounter = 1

    const handleClickAddTest = () => {
        testCounter++
        // Add a new row to the table
        tableRows.value.push({
            testName: null,
            mean: null,
            meanGroup: null
        })
        console.log(tableRows.value)
    }

    // Load QC data for editing
    const loadQcData = () => {
        if (!props.qcId) return
        
        isLoading.value = true
        
        fetch(`${import.meta.env.VITE_API_URL}/qc/${props.qcId}`, {
            method: 'GET',
            headers: { 'Content-Type': 'application/json' }
        })
        .then(response => {
            if (!response.ok) {
                throw new Error(`HTTP error: ${response.status}`)
            }
            return response.json()
        })
        .then(response => {
            if (response.ok && response.data) {
                // Populate form data with response
                const qcData = response.data

                /**
                * Processes QC data and initializes the form data
                * 
                * @remarks
                * The activityDate is converted to YYYY-MM-DD format (instead of using ISOString)
                * because this format is compatible with HTML date input fields, which require
                * date values in this specific format for proper display and interaction.
                * ISOString would include time and timezone information (e.g., 2023-05-15T08:30:00Z)
                * which is not suitable for date-only input fields.
                **/
                const date = new Date(qcData.activityDate);
                const convertedActivityDate = `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')}`;
                
                formData.value = {
                    anzId: qcData.anzId,
                    name: qcData.name,
                    periode: qcData.periode,
                    activityDate: new Date(qcData.activityDate)
                }

                
                // Populate table rows
                tableRows.value = qcData.details.map((detail: any) => ({
                    testName: detail.testCd,
                    mean: detail.mean,
                    meanGroup: detail.meanGroup
                }))
            } else {
                console.error('Error loading QC data:', response.error)
                statusMessage.value = 'Failed to load QC data'
                isSuccess.value = false
            }
        })
        .catch(error => {
            console.error('Error fetching QC data for editing:', error)
            statusMessage.value = 'Error loading data: ' + error.message
            isSuccess.value = false
        })
        .finally(() => {
            isLoading.value = false
        })
    }

    // Form submission handler using Promises
    const handleSubmit = (event: Event) => {
        event.preventDefault()

        console.log(formData.value.activityDate)
        
        if (!isFormValid.value) {
            // Display appropriate validation messages
            if (!formData.value.anzId || !formData.value.name || !formData.value.periode || !formData.value.activityDate) {
                statusMessage.value = 'Please fill in all required fields'
            } else if (tableRows.value.length === 0) {
                statusMessage.value = 'Please add at least one test'
            } else {
                statusMessage.value = 'Please ensure all test entries are valid'
            }
            
            isSuccess.value = false
            return
        }
        
        isLoading.value = true
        statusMessage.value = ''
        
        // Format data for API
        const details = tableRows.value.map(row => ({
            testCd: row.testName?.toUpperCase(),
            mean: row.mean !== null ? parseFloat(row.mean.toString()) : null,
            meanGroup: row.meanGroup !== null ? parseFloat(row.meanGroup.toString()) : null
        }))
        
        const payload = {
            anzId: formData.value.anzId,
            name: formData.value.name,
            periode: formData.value.periode,
            activityDate: formData.value.activityDate,
            details: details
        }
        
        // Determine API endpoint and method based on mode
        const url = isEditMode.value 
            ? `${import.meta.env.VITE_API_URL}/qc/${props.qcId}` 
            : `${import.meta.env.VITE_API_URL}/qc`
        
        const method = isEditMode.value ? 'PUT' : 'POST'
        
        // Send request using Promises
        fetch(url, {
            method: method,
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(payload)
        })
        .then(response => {
            if (!response.ok) {
                return response.json().then(data => {
                    throw new Error(data.message || 'Failed to submit data')
                })
            }
            return response.json()
        })
        .then(data => {
            // Success handling
            statusMessage.value = isEditMode.value 
                ? 'Data updated successfully!' 
                : 'Data submitted successfully!'
            isSuccess.value = true
            
            // if (!isEditMode.value) {
            //     // Only reset form for new records
            //     resetForm()
            // }
        })
        .catch(error => {
            // Error handling
            statusMessage.value = error.message || 'An error occurred'
            isSuccess.value = false
            console.error('Submission error:', error)
        })
        .finally(() => {
            isLoading.value = false
        })
    }
    
    // Reset form function
    const resetForm = () => {
        formData.value = {
            anzId: '',
            name: '',
            periode: '',
            activityDate: new Date()
        }
        tableRows.value = []
        testCounter = 0
    }

    // Handle selecting analyzer from dropdown
    const handleAnalyzerSelect = (value: string) => {
        formData.value.anzId = value
    }

    const goBack = () => {
        window.history.back()
    }
</script>

<template>
    <form id="form" @submit="handleSubmit">
        <div>
            <div class="mt-5">
                <BackButton @click="goBack" class="ms-20">Back</BackButton>
                <div class="text-center">
                    <h1 class="text-2xl font-semibold">{{ formTitle }}</h1>
                </div>
            </div>
            
            <!-- Status message -->
            <div v-if="statusMessage" 
                 :class="['text-center p-2 rounded mx-20 mb-4', isSuccess ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700']">
                {{ statusMessage }}
            </div>
            
            <!-- Loading indicator -->
            <div v-if="isLoading && isEditMode" class="text-center p-2">
                <span class="text-gray-600">Loading data...</span>
            </div>
            
            <div class="flex flex-wrap justify-around items-baseline pt-5 mx-20">
                <div class="w-full xl:w-1/2 mb-6 md:mb-0">
                    <div class="w-9/12 pb-4">
                        <label for="analyzer-selector" class="font-medium text-nowrap">Analyzer</label>
                        <select 
                            id="analyzer-selector"
                            name="analyzer-selector" 
                            v-model="formData.anzId" 
                            class="border border-gray-300 rounded-md p-2 w-full"
                            :disabled="isLoadingAnalyzers"
                        >
                            <option v-if="isLoadingAnalyzers" value="" disabled selected>Loading analyzers...</option>
                            <option v-else value="" disabled selected>Select an analyzer</option>
                            <option v-for="anz in analyzers" :key="anz.value" :value="anz.value">{{ anz.name }}</option>
                        </select>
                    </div>
                    <div class="w-9/12 pb-4">
                        <label for="pme-selector" class="font-medium text-nowrap">PME Name</label>
                        <select
                            id="pme-selector"
                            name="pme-selector"
                            v-model="formData.name"
                            class="border border-gray-300 rounded-md p-2 w-full"
                            :disabled="isLoadingPmes"
                        >
                            <option v-if="isLoadingPmes" value="" disabled selected>Loading PMEs...</option>
                            <option v-else value="" disabled selected>Select a PME</option>
                            <option v-for="pme in pmes" :key="pme.value" :value="pme.value">{{ pme.name }}</option>
                        </select>
                    </div>
                    <div class="w-9/12 pb-4">
                        <label for="periode-input" class="font-medium text-nowrap">Periode</label>
                        <input 
                            type="text" 
                            id="periode-input"
                            name="periode-input" 
                            v-model="formData.periode" 
                            class="border border-gray-300 rounded-md p-2 w-full"  
                            placeholder="e.g. 2025-01"
                        />
                    </div>
                    <div class="w-9/12 pb-4">
                        <label for="activity-date" class="font-medium text-nowrap">Activity Date</label>
                        <input 
                            type="date" 
                            id="activity-date"
                            name="activityDate" 
                            v-model="formData.activityDate" 
                            class="border border-gray-300 rounded-md p-2 w-full"
                        />
                    </div>
                </div>
                <div id="qc-details-table" class="w-full xl:w-1/2 sm:pt-10">
                    <TableInput 
                        :tableRows="tableRows"
                        @update:tableRows="newRows => tableRows = newRows" 
                    />
                    <div class="py-1.5 w-full">
                        <AddTestButton @click="handleClickAddTest" />
                    </div>
                </div>
            </div>
            <div class="flex justify-center items-center mt-20">
                <button
                    v-if="isEditMode"
                    id="cancel-button"
                    class="bg-gray-300 text-gray-700 shadow-lg py-1.5 px-12 rounded-lg mt-5 mr-5 hover:bg-gray-400"
                    type="button"
                    @click="goBack"
                >
                    Cancel
                </button>
                <button
                    v-if="!isEditMode"
                    id="clear-button"
                    class="bg-red-300 text-white shadow-lg py-1.5 px-12 rounded-lg mt-5 mr-5 hover:bg-red-400"
                    type="reset"
                    @click="resetForm"
                >
                    Clear
                </button>
                <button 
                    id="submit-button"
                    class=" shadow-lg py-1.5 px-12 rounded-lg mt-5" 
                    type="submit"
                    :disabled="isLoading || !isFormValid"
                    :class="{
                        'bg-blue-400 text-white hover:bg-blue-500': !isLoading || isFormValid,
                        'bg-gray-200 text-black opacity-80 cursor-not-allowed': isLoading || !isFormValid
                    }"
                >
                    {{ isLoading ? 'Processing...' : (isEditMode ? 'Update' : 'Submit') }}
                </button>
            </div>
        </div>
    </form>
</template>