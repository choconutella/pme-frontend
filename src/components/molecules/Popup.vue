<script setup lang="ts">
    const props = defineProps({
        show: {
            type: Boolean,
            default: false
        },
        title: {
            type: String,
            default: 'Confirm Deletion'
        },
        message: {
            type: String,
            default: 'Are you sure you want to delete this item? This action cannot be undone.'
        }
    })

    const emit = defineEmits(['cancel', 'confirm'])

    const onCancel = () => {
        emit('cancel')
    }

    const onConfirm = () => {
        emit('confirm')
    }
</script>

<template>
    <transition name="fade">
        <div v-if="show" class="popup-overlay" @click.self="onCancel">
            <div class="popup-container">
                <div class="popup-header">
                    <h3>{{ title }}</h3>
                    <button class="close-button" @click="onCancel">&times;</button>
                </div>
                <div class="popup-content">
                    {{ message }}
                </div>
                <div class="popup-actions">
                    <button class="bg-gray-100 hover:bg-gray-200" @click="onCancel">Cancel</button>
                    <button class="text-white bg-red-500 hover:bg-red-700" @click="onConfirm">Delete</button>
                </div>
            </div>
        </div>
    </transition>
</template>



<style scoped>
    .popup-overlay {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.5);
        display: flex;
        justify-content: center;
        align-items: center;
        z-index: 1000;
    }

    .popup-container {
        background-color: #fff;
        border-radius: 8px;
        box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
        width: 90%;
        max-width: 500px;
        overflow: hidden;
    }

    .popup-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 15px 20px;
        background-color: #f5f5f5;
        border-bottom: 1px solid #e0e0e0;
    }

    .popup-header h3 {
        margin: 0;
        font-size: 18px;
        color: #333;
    }

    .close-button {
        background: none;
        border: none;
        font-size: 22px;
        cursor: pointer;
        color: #666;
    }

    .popup-content {
        padding: 20px;
        color: #555;
    }

    .popup-actions {
        padding: 15px 20px;
        display: flex;
        justify-content: flex-end;
        gap: 10px;
        border-top: 1px solid #e0e0e0;
    }

    button {
        padding: 8px 16px;
        border-radius: 4px;
        cursor: pointer;
        font-size: 14px;
        border: none;
    }

    .fade-enter-active, .fade-leave-active {
        transition: opacity 0.3s;
    }

    .fade-enter-from, .fade-leave-to {
        opacity: 0;
    }
</style>