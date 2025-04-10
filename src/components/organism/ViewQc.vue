<script setup lang="ts">
    import NewButton from '@/components/molecules/NewButton.vue'
    import EditButton from '@/components/molecules/EditButton.vue'
    import DeleteButton from '@/components/molecules/DeleteButton.vue'
    import Popup from '@/components/molecules/Popup.vue'
    import { useRouter } from 'vue-router'
    import { ref, onMounted, computed } from 'vue'

    const router = useRouter()
    const search = ref<HTMLInputElement | null>(null)
    const searchQuery = ref('')

    // State for delete confirmation popup
    const showDeletePopup = ref(false)
    const qcToDelete = ref<QcData | null>(null)

    interface Analyzer {
        code: string
        name: string
    }

    interface QcData {
        id: number
        anz: Analyzer
        name: string
        periode: string
        activityDate: string
    }

    const qcs = ref<QcData[]>([])

    const fetchQcData = () => {
        fetch(`${import.meta.env.VITE_API_URL}/qc`,
            {
                method: "GET",
                headers: { "Content-Type": "application/json" },
            },
        )
            .then(response => {
                if (!response.ok) {
                    throw new Error(`HTTP error: ${response.status}`)
                }
                return response.json()
            })
            .then((data) => {
                if (data.ok) {
                    qcs.value = data.data;
                } else {
                    console.error('Error fetching QC data:', data.error)
                }
            })
            .catch(error => {
                console.error('Failed to fetch QC data:', error)
            })
    }

    const filteredQcs = computed(() => {
        if (!searchQuery.value) return qcs.value
        return qcs.value.filter(qc => qc.anz.name.toLowerCase().includes(searchQuery.value.toLowerCase()))
    })

    // Function to show delete confirmation
    const confirmDelete = (qc: QcData) => {
        qcToDelete.value = qc
        showDeletePopup.value = true
    }

    // Function to handle delete cancellation
    const cancelDelete = () => {
        showDeletePopup.value = false
        qcToDelete.value = null
    }

    const handleEdit = (qc: QcData) => {
        router.push(`/qc/${qc.id}`)
    }

    const handleDelete = () => {
        if (!qcToDelete.value) {
            console.error('No QC selected for deletion')
            return
        }

        const id = qcToDelete.value.id

        fetch(`${import.meta.env.VITE_API_URL}/qc/${id}`, {
            method: 'DELETE',
            headers: {
                'Content-Type': 'application/json'
            }
        })
        .then(response => {
            console.log('Delete response status:', response.status)
            if (!response.ok) {
                throw new Error(`HTTP error: ${response.status}`)
            }
            return response.json()
        })
        .then(data => {
            qcs.value = qcs.value.filter(qc => qc.id !== id)
            showDeletePopup.value = false
            qcToDelete.value = null

        })
        .catch(error => {
            console.error('Error deleting QC data:', error)
        })
    }

    onMounted(() => {
        fetchQcData()
        
        document.addEventListener('keydown', (event) => {
            if ((event.ctrlKey || event.metaKey) && event.key === 'k') {
                event.preventDefault() // Prevent default browser behavior
                search.value?.focus()
            }
        })
    })
</script>

<template>
    <div>
        <div class="flex justify-center items-center text-2xl font-semibold py-7">
            <h1>List QC PME</h1>
        </div>
        <div class="py-4 flex justify-end">
            <div class="w-3/12">
                <input 
                    id="search-input"
                    ref="search"
                    v-model="searchQuery"
                    type="text" 
                    class="w-full border border-gray-300 rounded-md px-3 py-1" 
                    placeholder="CTRL+K Filter By Analyzer" />
            </div>
            <div id="back-button" class="ps-4">
                <NewButton @click="router.push('/qc/new')" />
            </div>
        </div>
        <div>
            <table class="w-full border-collapse border-y border-gray-300">
                <thead>
                    <tr>
                        <th class="w-1/12">No.</th>
                        <th class="text-left">Analyzer</th>
                        <th class="w-2/12 text-left">Name</th>
                        <th class="w-2/12 text-left">Periode</th>
                        <th class="w-2/12">Activity Date</th>
                        <th class="w-2/12">Action</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-if="filteredQcs.length === 0">
                        <td colspan="6" class="text-center py-4 text-gray-500">
                            No results found
                        </td>
                    </tr>
                    <tr v-for="qc, index in filteredQcs" :key="qc.id">
                        <td class="text-center py-2">{{ index + 1 }}</td>
                        <td class="py-2">{{ qc.anz.name }}</td>
                        <td class="py-2">{{ qc.name }}</td>
                        <td class="py-2">{{ qc.periode }}</td>
                        <td class="text-center py-2">{{ new Date(qc.activityDate).toLocaleDateString() }}</td>
                        <td class="flex justify-center items-center py-2">    
                            <EditButton class="px-2" @click="handleEdit(qc)" />
                            <DeleteButton class="px-2" @click="confirmDelete(qc)" />
                        </td>
                    </tr>
                </tbody>    
            </table>
        </div>

        <!-- Delete Confirmation Popup -->
        <Popup 
            :show="showDeletePopup"
            :title="`Delete ${qcToDelete?.anz?.name} - ${qcToDelete?.name}`"
            :message="`Are you sure you want to delete ${qcToDelete?.name} for ${qcToDelete?.anz?.name}? This action cannot be undone.`"
            @cancel="cancelDelete"
            @confirm="handleDelete"
        />
    </div>
</template>