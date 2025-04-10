<script setup lang="ts">
    import { ref, onMounted, watch } from 'vue'
    import { useRouter, useRoute } from 'vue-router'
    import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'
    // import { faList, faPlus, faBars, faXmark } from '@fortawesome/free-solid-svg-icons'

    const isOpen = ref(false)
    const router = useRouter()
    const route = useRoute()

    const toggleNav = () => {
        isOpen.value = !isOpen.value
    }

    const closeNav = () => {
        if (window.innerWidth < 1024) {
            isOpen.value = false
        }
    }

    // Track screen size to auto-open nav on large screens
    onMounted(() => {
        const checkScreenSize = () => {
            if (window.innerWidth >= 1024) {
            isOpen.value = true
            } else {
            isOpen.value = false
            }
        }
        
        // Call once on mount
        checkScreenSize()
        
        // Add event listener for resize
        window.addEventListener('resize', checkScreenSize)
        
        // Clean up
        return () => window.removeEventListener('resize', checkScreenSize)
    })

    // Close nav on route change on mobile
    watch(() => route.path, () => {
        if (window.innerWidth < 1024) {
            closeNav()
        }
    })

    const navItems = [
        { name: 'List QC PME', path: '/qc' },
        { name: 'New QC PME', path: '/qc/new' }
    ]
</script>

<template>
    <div class="relative">
        <!-- Toggle button for mobile -->
        <button 
            @click="toggleNav" 
            class="lg:hidden fixed z-20 top-20 left-4 bg-blue-400 text-white p-2 rounded-md"
            aria-label="Toggle navigation"
        >
            <FontAwesomeIcon :icon="isOpen ? faXmark : faBars" size="lg" />
        </button>

        <!-- Backdrop for mobile -->
        <div 
            v-if="isOpen && window.innerWidth < 1024" 
            class="fixed inset-0 bg-black bg-opacity-50 z-10"
            @click="toggleNav"
        ></div>

        <!-- Sidebar -->
        <aside 
            class="fixed top-0 left-0 z-10 h-full bg-blue-500 text-white transition-all duration-300 ease-in-out shadow-lg"
            :class="[
                isOpen ? 'translate-x-0' : '-translate-x-full',
                'lg:translate-x-0 lg:mt-16 lg:pt-10 pt-24 w-64'
            ]"
        >
            <div class="px-4 py-2">
                <h2 class="text-xl font-bold mb-6 text-center">QC Performance</h2>
                
                <nav>
                <ul class="space-y-2">
                    <li v-for="item in navItems" :key="item.path">
                    <router-link 
                        :to="item.path" 
                        class="flex items-center px-4 py-3 rounded-md transition-colors duration-200"
                        :class="route.path === item.path ? 'bg-blue-700' : 'hover:bg-blue-600'"
                    >
                        <!-- <FontAwesomeIcon :icon="item.icon" class="mr-3" /> -->
                        <span>{{ item.name }}</span>
                    </router-link>
                    </li>
                </ul>
                </nav>
            </div>
        </aside>
    </div>
</template>

<style scoped>
/* Add transition for smooth opening/closing */
.translate-x-0 {
  transform: translateX(0);
}
.-translate-x-full {
  transform: translateX(-100%);
}
</style>