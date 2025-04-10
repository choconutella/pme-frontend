<script setup lang="ts">
	const props = defineProps({
	modelValue: Boolean, // v-model binding (instead of isOpen)
	})

	const emit = defineEmits(['update:modelValue']) // Emits update event
</script>

<template>
	<nav :class="['sidebar', { closed: !modelValue }]">
		<button class="toggle-btn" @click="emit('update:modelValue', !modelValue)">
			{{ modelValue ? '❮' : '❯' }}
		</button>
		<ul>
			<li><a href="#">Dashboard</a></li>
			<li><a href="#">Reports</a></li>
			<li><a href="#">Settings</a></li>
			<li><a href="#">Logout</a></li>
		</ul>
	</nav>
</template>
