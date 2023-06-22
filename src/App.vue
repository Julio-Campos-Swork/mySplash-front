<template>
	<main>
		<v-container fluid>
			<v-row justify="space-evenly" class="mt-4 mb-4">
				<v-tooltip text="Set password">
					<template v-slot:activator="{ props }">
						<v-btn
							v-bind="props"
							variant="text"
							size="x-large"
							icon="mdi-account"
							@click="userDialog = true"
						></v-btn>
					</template>
				</v-tooltip>
				<!-- <span class="mdi mdi-account text-h3 ml-4 mt-4"> </span> -->
				<v-col cols="2">
					<p>My Unsplash</p>
					<p>devchallenges.io</p>
				</v-col>
				<v-col cols="6">
					<v-text-field
						v-model="searchLabel"
						variant="underlined"
						prepend-icon="mdi-magnify"
						clearable
						label="Search by label"
					></v-text-field>
				</v-col>
				<v-col cols="2">
					<v-btn color="green" rounded="lg" @click="uploadDialog = true"
						>Add a photo</v-btn
					>
				</v-col>
			</v-row>
		</v-container>
		<v-container fluid>
			<v-row>
				<v-col
					cols="4"
					v-if="!isSearching"
					v-for="images of listOfImages.images"
					:key="images.id"
				>
					<v-hover v-slot="{ isHovering, props }">
						<v-card
							max-width="350"
							:elevation="isHovering ? 12 : 2"
							v-bind="props"
							:class="{ 'on-hover': isHovering }"
						>
							<v-card-text>
								<v-img :src="images.url" :lazy-src="images.url">
									<template v-slot:placeholder>
          <v-row
            class="fill-height ma-0"
            align="center"
            justify="center"
          >
            <v-progress-circular
              indeterminate
              color="grey-lighten-5"
            ></v-progress-circular>
          </v-row>
        </template>
									<div class="button" v-if="isHovering">
										<v-btn
											@click="deleteItem(images)"
											color="red"
											rounded="lg"
											variant="outlined"
											>Delete</v-btn
										>
									</div>
									<div class="label" v-if="isHovering">
										{{ images.label }}
									</div>
								</v-img>
							</v-card-text>
						</v-card>
					</v-hover>
				</v-col>
				<v-col
					v-if="isSearching"
					cols="4"
					v-for="images of filteredImages"
					:key="images.id"
				>
					<v-hover v-slot="{ isHovering, props }">
						<v-card
							max-width="350"
							:elevation="isHovering ? 12 : 2"
							v-bind="props"
							:class="{ 'on-hover': isHovering }"
						>
							<v-card-text>
								<v-img :src="images.url" :lazy-src="images.url">
									<template v-slot:placeholder>
          <v-row
            class="fill-height ma-0"
            align="center"
            justify="center"
          >
            <v-progress-circular
              indeterminate
              color="grey-lighten-5"
            ></v-progress-circular>
          </v-row>
        </template>
									<div class="button" v-if="isHovering">
										<v-btn
											@click="deleteItem(images)"
											color="red"
											rounded="lg"
											variant="outlined"
											>Delete</v-btn
										>
									</div>
									<div class="label" v-if="isHovering">
										{{ images.label }}
									</div>
								</v-img>
							</v-card-text>
						</v-card>
					</v-hover>
				</v-col>
			</v-row>
		</v-container>

		<v-dialog v-model="uploadDialog" width="650">
			<v-card>
				<v-card-title class="text-h5 ml-4 mt-4">Add a new photo</v-card-title>
				<v-card-text>
					<v-row>
						<v-col>
							<v-label class="text-h6 text-black mb-2">Label</v-label>
							<v-text-field v-model="newLabel" variant="outlined" rounded></v-text-field>
						</v-col>
					</v-row>
					<v-row>
						<v-col>
							<v-label class="text-h6 text-black mb-2">Photo URL</v-label>
							<v-text-field
								v-model="newURL"
								variant="outlined"
								rounded="lg"
							></v-text-field>
						</v-col>
					</v-row>
				</v-card-text>
				<v-card-actions>
					<v-spacer></v-spacer>
					<v-btn variant="flat" rounded="lg" color="grey" size="x-large" @click="cancel()"
						>Cancel</v-btn
					>
					<v-btn
						variant="flat"
						rounded="lg"
						color="green"
						size="x-large"
						@click="uploadPhoto()"
						>Submit</v-btn
					>
				</v-card-actions>
			</v-card>
		</v-dialog>

		<!-- modal password -->

		<v-dialog v-model="userDialog" width="650">
			<v-card>
				<v-card-title class="text-h5 ml-4 mt-4">Set user password</v-card-title>
				<v-card-text>
					<v-col cols="12">
						<v-label class="text-h6 text-black mb-2">Password</v-label>
						<v-text-field
							v-model="setUserPassword"
							variant="outlined"
							rounded="lg"
							:append-icon="showPass ? 'mdi-eye' : 'mdi-eye-off'"
							:type="showPass ? 'text' : 'password'"
							@click:append="showPass = !showPass"
						></v-text-field>
					</v-col>
				</v-card-text>
				<v-card-actions>
					<v-spacer></v-spacer>
					<v-btn
						variant="flat"
						rounded="lg"
						color="grey"
						size="x-large"
						@click="userDialog = false"
						>Cancel</v-btn
					>
					<v-btn
						variant="flat"
						rounded="lg"
						color="green"
						size="x-large"
						@click="setPassword()"
						>Submit password</v-btn
					>
				</v-card-actions>
			</v-card>
		</v-dialog>

		<!-- modal Delete -->
		<v-dialog v-model="deleteDialog" width="650">
			<v-card>
				<v-card-title class="text-h5 ml-4 mt-4">Delete</v-card-title>
				<v-card-text>
					<v-col cols="12">
						<v-label class="text-h6 text-black mb-2"
							>Password (set password on user icon)</v-label
						>
						<v-text-field
							v-model="setUserPassword"
							variant="outlined"
							label="Enter your password"
							rounded="lg"
							:append-icon="showPass ? 'mdi-eye' : 'mdi-eye-off'"
							:type="showPass ? 'text' : 'password'"
							@click:append="showPass = !showPass"
						></v-text-field>
					</v-col>
				</v-card-text>
				<v-card-actions>
					<v-spacer></v-spacer>
					<v-btn
						variant="flat"
						rounded="lg"
						color="grey"
						size="x-large"
						@click="deleteDialog = false"
						>Cancel</v-btn
					>
					<v-btn
						variant="flat"
						rounded="lg"
						color="red"
						size="x-large"
						@click="confirmDelete()"
						>Confirm</v-btn
					>
				</v-card-actions>
			</v-card>
		</v-dialog>
		<!-- notification -->
		<v-snackbar v-model="notification" timeout="3000" location="top" :color="colorNot">
			{{ message }}
		</v-snackbar>
	</main>
</template>

<script setup>
	import { ref, reactive, watch } from 'vue'
	import axios from 'axios'
	const showPass = ref(false)
	const listOfImages = reactive({ images: [] })
	const filteredImages = ref([])
	const deleteDialog = ref(false)
	const itemToDelete = ref(null)
	const uploadDialog = ref(false)
	const userDialog = ref(false)
	const userPassword = ref('')
	const setUserPassword = ref('')
	const searchLabel = ref('')
	const newLabel = ref('')
	const newURL = ref('')
	const notification = ref(false)
	const colorNot = ref('')
	const message = ref('')
	// const BASE_URL = 'http://192.168.0.120/imgUploader/public/' //local
	const BASE_URL = 'https://desarrollogaren.000webhostapp.com/imgUploader/public/' //host
	//
	const getAllImages = async () => {
		searchLabel.value = ''
		const response = await axios.get(`${BASE_URL}api/getAllImages`)
		const data = response.data
		// console.log({ data })
		data.images.forEach((element) => {
			const fileName = element.split('/')[7]
			const id = fileName.split('_')[0]
			const url = element
			const label = element.split('_')[1]

			// Verificar si el elemento ya existe en el array
			const existingElement = listOfImages.images.find((image) => image.id === id)
			if (!existingElement) {
				const newElement = { id, url, label, fileName }
				listOfImages.images.push(newElement)
			}
		})

		// Ordenar las imágenes por tiempo (campo time en el nombre)
		listOfImages.images.sort((a, b) => {
			const timeA = parseInt(a.url.split('_')[2])
			const timeB = parseInt(b.url.split('_')[2])
			return timeB - timeA // Orden descendente, de más reciente a más antiguo
		})
	}
	getAllImages()

	const uploadPhoto = async () => {
		if (newLabel.value == '' || newURL.value == '') {
			notification.value = true
			message.value = 'Both fields are required'
			colorNot.value = 'yellow'
			return
		}
		const isValidURL = validateURL(newURL.value)
		if (!isValidURL) {
			notification.value = true
			message.value = 'Invalid URL'
			colorNot.value = 'yellow'
			return
		}
		const isImageValid = await checkImageValidity(newURL.value)
		if (!isImageValid) {
			notification.value = true
			message.value = 'URL is not a image'
			colorNot.value = 'red'
			return
		}

		const datos = new FormData()
		datos.append('imageURL', newURL.value)
		datos.append('label', newLabel.value)
		const response = await axios.post(`${BASE_URL}api/uploadIMGByUrl`, datos)
		const data = response.data
		// console.log({ data })
		if (data.status) {
			await getAllImages()
			uploadDialog.value = false
		} else {
			notification.value = true
			message.value = 'Error on save'
			colorNot.value = 'red'
		}
	}

	// validations
	const validateURL = (url) => {
		const urlRegex = /^(ftp|http|https):\/\/[^ "]+$/
		return urlRegex.test(url)
	}

	const checkImageValidity = async (url) => {
		try {
			const response = await axios.head(url)
			// console.log({ response })
			const contentType = response.headers['content-type']
			return contentType.includes('image/') || contentType.includes('binary/octet-stream')
		} catch (error) {
			return false
		}
	}

	const cancel = () => {
		uploadDialog.value = false
		newLabel.value = ''
		newURL.value = ''
	}

	const setPassword = () => {
		userPassword.value = setUserPassword.value
		setUserPassword.value = ''
		userDialog.value = false
		showPass.value = false
	}

	const deleteItem = async (image) => {
		deleteDialog.value = true
		itemToDelete.value = image
	}

	const confirmDelete = async () => {
		if (userPassword.value != setUserPassword.value) {
			notification.value = true
			message.value = 'Wrong password! Try again'
			colorNot.value = 'red'
			return
		}
		const PATH = `${BASE_URL}api/deleteImage`
		const datos = new FormData()
		datos.append('fileName', itemToDelete.value.fileName)
		const response = await axios.post(PATH, datos)
		const data = response.data
		// console.log({ data })
		if (!data.status) {
			notification.value = true
			message.value = 'Error deleting image'

			colorNot.value = 'red'
		}
		const index = listOfImages.images.findIndex(
			(element) => element.id == itemToDelete.value.id
		)
		if (index != -1) listOfImages.images.splice(index, 1)
		notification.value = true
		message.value = 'Delete successfully'
		colorNot.value = 'green'
		deleteDialog.value = false
	}

	const isSearching = ref(false)
	const filterImagesByLabel = () => {
		if (searchLabel.value.trim() === '') {
			isSearching.value = false
			// Si el campo de búsqueda está vacío, mostrar todas las imágenes sin filtrar
			filteredImages.value = listOfImages.images
		} else {
			// Filtrar las imágenes según el campo "label"
			isSearching.value = true
			filteredImages.value = listOfImages.images.filter((image) => {
				return image.label.toLowerCase().includes(searchLabel.value.toLowerCase())
			})
		}
	}
	watch(searchLabel, filterImagesByLabel)
</script>

<style scoped>
	.v-card:is(.on-hover) {
		opacity: 0.6;
	}

	.label {
		display: flex;
		justify-content: left;
		align-items: center;
		opacity: 1 !important;
		color: #000000;
		position: absolute;
		bottom: 0;
		left: 0;
		right: 0;
		padding: 8px;
		font-size: 18px;
		font-weight: 900;

		pointer-events: none;
		text-shadow: -1px -1px 0 #ffffff;
	}

	.button {
		display: flex;
		justify-content: right;
		align-items: center;
		opacity: 1 !important;
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		padding: 8px;
		font-size: 18px;
		font-weight: 900;
	}

	/* .v-card.on-hover .label {
  opacity: 1 !important;
} */
</style>
