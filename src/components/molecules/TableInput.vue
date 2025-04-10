<script setup lang="ts">
    import { ref } from 'vue'
    import TextBox from '@/components/atoms/TextBox.vue';

    // Define props to receive the rows from parent
    const props = defineProps<{
        tableRows: Array<{
            testName: string | null;
            mean: number | null;
            meanGroup: number | null;
        }>
    }>()

    // Define emit to allow parent to modify data
    const emit = defineEmits(['update:tableRows'])
    function updateTestNameRow(index: number, input: any) {
        const newRows = [...props.tableRows]
        newRows[index].testName = input.target.value
        emit('update:tableRows', newRows)
    }
    function updateMeanRow(index: number, input: any) {
        const newRows = [...props.tableRows]
        newRows[index].mean = input.target.value
        emit('update:tableRows', newRows)
    }
    function updateMeanGroupRow(index: number, input: any) {
        const newRows = [...props.tableRows]
        newRows[index].meanGroup = input.target.value
        emit('update:tableRows', newRows)
    }
</script>

<template>
    <table class="w-full border-separate border-spacing-1">
        <thead>
            <tr>
                <th class="border py-1.5 w-3/12 bg-blue-400">Test Code</th>
                <th class="border py-1.5 w-3/12 bg-blue-400">Mean</th>
                <th class="border py-1.5 w-3/12 bg-blue-400">Mean Group</th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="(row, index) in tableRows" :key="index">
                <td class="w-3/12">
                    <input 
                        id="test-input-{{ index }}"
                        type="text" 
                        :value="row.testName?.toUpperCase()" 
                        placeholder="e.g. HB" @input="value => updateTestNameRow(index, value)" 
                        class="border border-gray-300 rounded-md p-2 w-full"
                    />
                </td>
                <td class="w-3/12">
                    <input 
                        id="mean-input-{{ index }}"
                        type="text" 
                        :value="row.mean" 
                        placeholder="e.g. 10.4" 
                        @input="value => updateMeanRow(index, value)" 
                        class="border border-gray-300 rounded-md p-2 w-full"
                    />
                </td>
                <td class="w-3/12">
                    <input 
                        id="mean-group-input-{{ index }}"
                        type="text" 
                        :value="row.meanGroup" 
                        placeholder="e.g. 10.4" 
                        @input="value => updateMeanGroupRow(index, value)" 
                        class="border border-gray-300 rounded-md p-2 w-full"
                    />
                </td>
            </tr>
        </tbody>
    </table>
</template>