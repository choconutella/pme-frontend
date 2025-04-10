<script setup lang="ts">
    import Header from '@/components/organism/Header.vue'
    import FormQc from '@/components/organism/FormQc.vue'
    import Footer from '@/components/organism/Footer.vue'

    defineProps<{
        id?: string
    }>()
</script>

<template>
    <Header title="QC Performance" userName="User" />
    <FormQc :qcId="id" class="md:mx-5" />
    <Footer />
</template>
