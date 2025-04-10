<script setup lang="ts">
    import Header from '@/components/organism/Header.vue'
    import Footer from '@/components/organism/Footer.vue'
    import CalcQc from '@/components/organism/CalcQc.vue'
</script>

<template> 
    <Header title="QC Performance" userName="User" />
    <CalcQc class="mt-10 mx-5 mb-5" />
    <Footer />
</template>