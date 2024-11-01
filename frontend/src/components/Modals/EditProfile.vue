<template>
	<Dialog
		:options="{
			title: 'Edit your profile',
			size: 'xl',
			actions: [
				{
					label: 'Save',
					variant: 'solid',
					onClick: (close) => saveProfile(close),
				},
			],
		}"
	>
		<template #body-content>
			<div>
				<FileUploader
					v-if="!profile.image"
					:fileTypes="['image/*']"
					:validateFile="validateFile"
					@success="(file) => saveImage(file)"
				>
					<template v-slot="{ file, progress, uploading, openFileSelector }">
						<div class="mb-4">
							<Button @click="openFileSelector" :loading="uploading">
								{{
									uploading
										? `Uploading ${progress}%`
										: 'Upload a profile image'
								}}
							</Button>
						</div>
					</template>
				</FileUploader>
				<div v-else class="mb-4">
					<div class="text-xs text-gray-600 mb-1">
						{{ __('Profile Image') }}
					</div>
					<div class="flex items-center">
						<div class="border rounded-md p-2 mr-2">
							<FileText class="h-5 w-5 stroke-1.5 text-gray-700" />
						</div>
						<div class="text-base flex flex-col">
							<span>
								{{ profile.image.file_name }}
							</span>
							<span class="text-sm text-gray-500 mt-1">
								{{ getFileSize(profile.image.file_size) }}
							</span>
						</div>
						<X
							@click="removeImage()"
							class="bg-gray-200 rounded-md cursor-pointer stroke-1.5 w-5 h-5 p-1 ml-4"
						/>
					</div>
				</div>
				
				<!-- Basic Info Section -->
				<FormControl
					v-model="profile.first_name"
					:label="__('First Name')"
					class="mb-4"
				/>
				<FormControl
					v-model="profile.last_name"
					:label="__('Last Name')"
					class="mb-4"
				/>
				<FormControl
					v-model="profile.headline"
					:label="__('Headline')"
					class="mb-4"
				/>

				<!-- Social Links Section -->
				<div class="mb-4">
					<div class="text-sm font-medium text-gray-700 mb-3">{{ __('Social Links') }}</div>
					<FormControl
						v-model="profile.user_website"
						:label="__('Website')"
						placeholder="https://yourwebsite.com"
						class="mb-4"
					/>
					<FormControl
						v-model="profile.linkedin_1"
						:label="__('LinkedIn')"
						placeholder="https://linkedin.com/in/yourprofile"
						class="mb-4"
					/>
					<FormControl
						v-model="profile.instagram"
						:label="__('Instagram')"
						placeholder="https://instagram.com/yourhandle"
						class="mb-4"
					/>
					<FormControl
						v-model="profile.crunchbase"
						:label="__('Crunchbase')"
						placeholder="https://crunchbase.com/person/yourprofile"
						class="mb-4"
					/>
				</div>

				<!-- Bio Section -->
				<FormControl type="textarea" v-model="profile.bio" :label="__('Bio')" />
			</div>
		</template>
	</Dialog>
</template>

<script setup>
import {
	Dialog,
	FormControl,
	FileUploader,
	Button,
	createResource,
} from 'frappe-ui'
import { reactive, watch, defineModel } from 'vue'
import { FileText, X } from 'lucide-vue-next'
import { getFileSize, showToast } from '@/utils'

const reloadProfile = defineModel('reloadProfile')

const props = defineProps({
	profile: {
		type: Object,
		required: true,
	},
})

const profile = reactive({
	first_name: '',
	last_name: '',
	headline: '',
	bio: '',
	image: '',
	user_website: '',
	linkedin_1: '',
	instagram: '',
	crunchbase: ''
})

const imageResource = createResource({
	url: 'lms.lms.api.get_file_info',
	makeParams(values) {
		return {
			file_url: values.image,
		}
	},
	auto: false,
	onSuccess(data) {
		profile.image = data
	},
})

const updateProfile = createResource({
	url: 'frappe.client.set_value',
	makeParams(values) {
		return {
			doctype: 'User',
			name: props.profile.data.name,
			fieldname: {
				first_name: profile.first_name,
				last_name: profile.last_name,
				headline: profile.headline,
				bio: profile.bio,
				user_image: profile.image?.file_url,
				user_website: profile.user_website,
				linkedin_1: profile.linkedin_1,
				instagram: profile.instagram,
				crunchbase: profile.crunchbase
			},
		}
	},
	onSuccess(data) {
		props.profile.data = data
	},
})

const saveProfile = (close) => {
	updateProfile.submit(
		{},
		{
			onSuccess() {
				close()
				reloadProfile.value.reload()
			},
			onError(err) {
				showToast('Error', err.messages?.[0] || err, 'x')
			},
		}
	)
}

const validateFile = (file) => {
	let extension = file.name.split('.').pop().toLowerCase()
	if (!['jpg', 'jpeg', 'png'].includes(extension)) {
		return 'Only image file is allowed.'
	}
}

const saveImage = (file) => {
	profile.image = file
}

const removeImage = () => {
	profile.image = null
}

watch(
	() => props.profile.data,
	(newVal) => {
		if (newVal) {
			profile.first_name = newVal.first_name || ''
			profile.last_name = newVal.last_name || ''
			profile.headline = newVal.headline || ''
			profile.bio = newVal.bio || ''
			profile.user_website = newVal.user_website || ''
			profile.linkedin_1 = newVal.linkedin_1 || ''
			profile.instagram = newVal.instagram || ''
			profile.crunchbase = newVal.crunchbase || ''
			if (newVal.user_image) imageResource.submit({ image: newVal.user_image })
		}
	}
)
</script>