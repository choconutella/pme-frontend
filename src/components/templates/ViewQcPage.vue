<script setup lang="ts">
    import Header from '@/components/organism/Header.vue'
    import ViewQc from '@/components/organism/ViewQc.vue'
    import Footer from '@/components/organism/Footer.vue'
</script>

<template>
    <Header title="QC Performance" userName="User" />
    <ViewQc class="md:mx-10 xl:mx-40" />
    <Footer />
</template>