<script setup lang="ts">
    import Edit from '@/components/atoms/Edit.vue'
    defineEmits(['click'])
</script>

<template>
    <button class="cursor-pointer" type="button" @click="$emit('click')">
        <Edit role="button"/>
        <p class="text-sm font-light">Edit</p>
    </button>
</template>