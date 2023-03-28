<template>
	<div>
		<v-form class="my-4" @submit.prevent="uploadFile">
			<v-file-input
				v-model="file"
				:loading="isUploading"
				:placeholder="$t('common.uploadYourFiles')"
				:label="$t('common.attachYourFiles')"
				accept=".csv"
				show-size
				prepend-icon="mdi-paperclip"
			>
				<template v-slot:selection="{ text }">
					<v-chip label color="">
						{{ text }}
					</v-chip>
				</template>
			</v-file-input>

			<div class="my-2" v-if="file">
				<v-btn class="mr-1" type="submit" color="primary" small>
					{{ $t('common.uploadFile') }}
				</v-btn>

				<v-btn class="ml-1" color="error" small @click="resetFile">
					{{ $t('common.removeFile') }}
				</v-btn>
			</div>
		</v-form>

		<!-- <v-snackbar v-model="errorSnackbar" timeout="7000">
			<p class="my-0 font-weight-medium" v-html="snackbarMessage"></p>

			<p class="my-0 text-caption font-weight-light">
				{{ $t('errorMessage.checkDevLogs') }}
			</p>

			<template v-slot:action="{ attrs }">
				<v-btn color="pink" text v-bind="attrs" @click="errorSnackbar = false">
					{{ $t('common.close') }}
				</v-btn>
			</template>
		</v-snackbar> -->
	</div>
</template>

<script lang="ts">
import Vue from 'vue';

import axios, { AxiosRequestConfig } from 'axios';

type Data = {
	file: File | null;
	isUploading: boolean;
	errorSnackbar: boolean;
	snackbarMessage: string;
	authorizationToken: string | null;
};

export default Vue.extend({
	name: 'CSVFileUploader',
	props: {
		url: String,
	},
	data(): Data {
		return {
			file: null,
			isUploading: false,
			errorSnackbar: false,
			snackbarMessage: '',
			authorizationToken: localStorage.getItem('authorization_token'),
		};
	},
	methods: {
		async fetchPresignedS3Url(url: string, fileName: string) {
			console.log(
				'[fetchPresignedS3Url] authorizationToken: ',
				this.authorizationToken
			);

			try {
				const requestConfig: AxiosRequestConfig = {
					method: 'GET',
					url,
					params: {
						name: encodeURIComponent(fileName),
					},
				};
				if (this.authorizationToken) {
					requestConfig.headers = {
						Authorization: `Basic ${this.authorizationToken}`,
					};
				}
				const result = await axios(requestConfig);

				const data = result.data;
				return data;
			} catch (error) {
				console.log('[fetchPresignedS3Url] error:', error.response); // this is the main part. Use the response property from the error object
				this.showErrorSnackbar(error.response.data.message);
				return null;
			}
		},
		async uploadFileBy(url: string, file: File) {
			const destUrl = await this.fetchPresignedS3Url(url, file.name);
			console.log('[uploadFileBy] destUrl: ', destUrl);

			if (destUrl) {
				console.info('[uploadFileBy] Uploading to: ', destUrl?.url);

				// save
				const result = await fetch(destUrl?.url, {
					method: 'PUT',
					body: file,
				});

				console.info('[uploadFileBy] Result: ', result);

				return result;
			}
		},
		showUploadingStatus() {
			this.isUploading = true;
		},
		hideUploadingStatus() {
			this.isUploading = false;
		},
		showErrorSnackbar(msg: string, reason = 'error') {
			const message = this.$t(msg, {
				reason,
			});
			this.$store.dispatch('snackbar/showErrorSnackbar', { message });
		},
		async uploadFile() {
			this.showUploadingStatus();

			try {
				await this.uploadFileBy(this.url, this.file as File);
			} catch (e) {
				this.showErrorSnackbar(e.message);
			} finally {
				this.resetFile();
				this.hideUploadingStatus();
			}
		},
		resetFile() {
			this.file = null;
		},
	},
});
</script>
