<template>
	<div v-show="isErrorSnackbar">
		<v-error-snackbar
			:is-visible="snackbarData.visible"
			:message="snackbarData.message"
		/>
	</div>
</template>

<script lang="ts">
import Vue from 'vue';

import { SnackbarType } from '@/store/snackbar';
import type { SnackbarState } from '@/store/snackbar';

import VErrorSnackbar from './ui/ErrorSnackbar.vue';

export default Vue.extend({
	components: { VErrorSnackbar },
	computed: {
		snackbarData(): SnackbarState {
			return this.$store.state.snackbar;
		},
		isErrorSnackbar() {
			const snackbarData: SnackbarState = this.snackbarData;
			return snackbarData.type === SnackbarType.Error;
		},
	},
});
</script>
