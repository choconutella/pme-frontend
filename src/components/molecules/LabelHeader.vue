<script setup lang="ts">
    import Label from '@/components/atoms/Label.vue'

    const props = defineProps<{
        label: string
    }>()
</script>

<template>
    <div class="bg-blue-400 text-white shadow-lg py-1.5 sm:px-12 md:px-12 lg:px-10 xl:px-18 2xl:px-26 rounded-lg">
        <Label :label="label" />
    </div>
</template>