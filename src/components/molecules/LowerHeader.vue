<script setup lang="ts">
    import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'
    import { faBars } from '@fortawesome/free-solid-svg-icons'

    const props = defineProps<{
        userName: string
    }>()

</script>

<template>
    <div class="bg-blue-400 py-5 ps-5 flex justify-between items-center">
        <FontAwesomeIcon :icon="faBars" class="text-white" />
        <div class="pe-5 text-xl text-white font-semibold">Good Morning, {{ userName }}</div>
    </div>
</template>