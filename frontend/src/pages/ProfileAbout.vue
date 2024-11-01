<template>
	<div class="mt-7 mb-10 grid grid-cols-1 md:grid-cols-3 gap-6">
		<!-- Education Details Column - Increased width -->
		<div class="md:col-span-1 max-w-[108%]">
			<h2 class="mb-3 text-lg font-semibold text-gray-900">
				{{ __('Education') }}
			</h2>
			<div class="space-y-3">
				<!-- University and Program Section -->
				<div class="space-y-1">
					<div v-if="profile.data.university" class="text-base">
						<div>{{ profile.data.university }}</div>
					</div>
					<div v-if="profile.data.program" class="text-base">
						<div class="text-gray-700">{{ profile.data.program }}</div>
					</div>
				</div>
				
				<!-- Separator -->
				<hr class="border-gray-200 my-3" v-if="profile.data.major || profile.data.concentration" />
				
				<!-- Major and Concentration Section -->
				<div class="space-y-3">
					<div v-if="profile.data.major" class="text-base">
						<div class="font-medium text-gray-500">{{ __('Major(s)') }}</div>
						<div>{{ profile.data.major }}</div>
					</div>
					<div v-if="profile.data.concentration" class="text-base">
						<div class="font-medium text-gray-500">{{ __('Concentration') }}</div>
						<div>{{ profile.data.concentration }}</div>
					</div>
				</div>
			</div>
		</div>

		<!-- Rest of the template remains exactly the same -->
		<!-- About Section Column -->
		<div class="md:col-span-2">
			<h2 class="mb-3 text-lg font-semibold text-gray-900">
				{{ __('About') }}
			</h2>
			<div
				v-if="profile.data.bio"
				v-html="profile.data.bio"
				class="ProseMirror prose prose-table:table-fixed prose-td:p-2 prose-th:p-2 prose-td:border prose-th:border prose-td:border-gray-300 prose-th:border-gray-300 prose-td:relative prose-th:relative prose-th:bg-gray-100 prose-sm max-w-none !whitespace-normal"
			></div>
			<div v-else class="text-gray-700 text-sm italic">
				{{ __('No introduction') }}
			</div>
		</div>
	</div>

	<!-- Badges Section remains unchanged -->
	<div class="mt-7 mb-10" v-if="badges.data?.length">
		<h2 class="mb-3 text-lg font-semibold text-gray-900">
			{{ __('Achievements') }}
		</h2>
		<div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-5 gap-4">
			<div v-for="badge in badges.data">
				<Popover trigger="hover" :leaveDelay="Number(0.01)">
					<template #target>
						<div class="relative">
							<img
								:src="badge.badge_image"
								:alt="badge.badge"
								class="h-[80px]"
							/>
							<div
								v-if="badge.count > 1"
								class="flex items-end bg-gray-100 p-2 text-xs font-semibold rounded-full absolute right-0 bottom-0"
							>
								<span>
									<X class="w-3 h-3" />
								</span>
								{{ badge.count }}
							</div>
						</div>
					</template>
					<template #body-main>
						<div class="w-[250px] text-base">
							<img
								:src="badge.badge_image"
								:alt="badge.badge"
								class="bg-gray-100 rounded-t-md h-[200px] mx-auto"
							/>
							<div class="p-5">
								<div class="text-2xl font-semibold mb-2">
									{{ badge.badge }}
								</div>
								<div class="leading-5 mb-4">
									{{ badge.badge_description }}
								</div>
								<div class="flex flex-col mb-4">
									<span class="text-xs text-gray-700 font-medium mb-1">
										{{ __('Issued on') }}:
									</span>
									{{ dayjs(badge.issued_on).format('DD MMM YYYY') }}
								</div>
								<div class="flex flex-col">
									<span class="text-xs text-gray-700 font-medium mb-1">
										{{ __('Share on') }}:
									</span>
									<div class="flex items-center space-x-2">
										<Button
											variant="outline"
											size="sm"
											@click="shareOnSocial(badge, 'LinkedIn')"
										>
											<template #prefix>
												<LinkedinIcon class="h-3 w-3 text-gray-700" />
											</template>
											<span class="text-xs">
												{{ __('LinkedIn') }}
											</span>
										</Button>
										<Button
											variant="outline"
											size="sm"
											@click="shareOnSocial(badge, 'Twitter')"
										>
											<template #prefix>
												<Twitter class="h-3 w-3 text-gray-700" />
											</template>
											<span class="text-xs">
												{{ __('Twitter') }}
											</span>
										</Button>
									</div>
								</div>
							</div>
						</div>
					</template>
				</Popover>
			</div>
		</div>
	</div>
</template>

<script setup>
import { inject } from 'vue'
import { createResource, Popover, Button } from 'frappe-ui'
import { X, LinkedinIcon, Twitter } from 'lucide-vue-next'
import { sessionStore } from '@/stores/session'

const dayjs = inject('$dayjs')
const { branding } = sessionStore()

const props = defineProps({
	profile: {
		type: Object,
		required: true,
	},
})

const badges = createResource({
	url: 'frappe.client.get_list',
	params: {
		doctype: 'LMS Badge Assignment',
		fields: ['name', 'badge', 'badge_image', 'badge_description', 'issued_on'],
		filters: {
			member: props.profile.data.name,
		},
	},
	auto: true,
	transform(data) {
		let finalBadges = []
		let groupedBadges = Object.groupBy(data, ({ badge }) => badge)
		for (let badge in groupedBadges) {
			let badgeData = groupedBadges[badge][0]
			badgeData.count = groupedBadges[badge].length
			finalBadges.push(badgeData)
		}
		return finalBadges
	},
})

const shareOnSocial = (badge, medium) => {
	let shareUrl
	const url = encodeURIComponent(
		`${window.location.origin}/lms/badges/${badge.badge}/${props.profile.data?.email}`
	)
	const summary = `I am happy to announce that I earned the ${
		badge.badge
	} badge on ${dayjs(badge.issued_on).format('DD MMM YYYY')} at ${
		branding.data?.app_name
	}.`

	if (medium == 'LinkedIn')
		shareUrl = `https://www.linkedin.com/shareArticle?mini=true&url=${url}&text=${summary}`
	else if (medium == 'Twitter')
		shareUrl = `https://twitter.com/intent/tweet?text=${summary}&url=${url}`

	window.open(shareUrl, '_blank')
}
</script>