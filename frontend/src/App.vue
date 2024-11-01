<template>
	<template v-if="isAuthenticated">
		<Layout>
			<router-view />
		</Layout>
		<Dialogs />
		<Toasts />
	</template>
</template>

<script setup>
import { Toasts } from 'frappe-ui'
import { Dialogs } from '@/utils/dialogs'
import { computed, onMounted, onUnmounted, watch } from 'vue'
import { useScreenSize } from './utils/composables'
import DesktopLayout from './components/DesktopLayout.vue'
import MobileLayout from './components/MobileLayout.vue'
import { stopSession } from '@/telemetry'
import { init as initTelemetry } from '@/telemetry'
import { usersStore } from '@/stores/user'
import { sessionStore } from '@/stores/session'
import { useRouter } from 'vue-router'

const router = useRouter()
const screenSize = useScreenSize()
const { userResource } = usersStore()
const { isLoggedIn } = sessionStore()

const isAuthenticated = computed(() => {
	return isLoggedIn && userResource.data
})

// Watch for authentication status changes
watch(
	isAuthenticated,
	(newValue) => {
		if (!newValue && router.currentRoute.value.path !== '/login') {
			router.push('/login')
		}
	},
	{ immediate: true }
)

const Layout = computed(() => {
	if (screenSize.width < 640) {
		return MobileLayout
	} else {
		return DesktopLayout
	}
})

onMounted(async () => {
	if (!userResource.data) return
	await initTelemetry()
})

onUnmounted(() => {
	stopSession()
})
</script>