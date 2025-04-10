<script setup lang="ts">
    import Logo from '@/components/atoms/Logo.vue'
    import Title from '@/components/atoms/Title.vue'

    const props = defineProps<{
        title: string
    }>()
    
</script>

<template>
    <div class="flex justify-between items-center p-5 w-full">
        <Logo />
        <Title :title=title />
    </div>
</template>

