<!--
  - @copyright Copyright (c) 2018 René Gieling <github@dartcafe.de>
  -
  - @author René Gieling <github@dartcafe.de>
  -
  - @license GNU AGPL version 3 or any later version
  -
  - This program is free software: you can redistribute it and/or modify
  - it under the terms of the GNU Affero General Public License as
  - published by the Free Software Foundation, either version 3 of the
  - License, or (at your option) any later version.
  -
  - This program is distributed in the hope that it will be useful,
  - but WITHOUT ANY WARRANTY; without even the implied warranty of
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program.  If not, see <http://www.gnu.org/licenses/>.
  -
  -->

<template>
	<div class="user_settings">
		<NcCheckboxRadioSwitch :checked.sync="legalTermsInEmail" type="switch">
			{{ t('polls', 'Add terms links also to the email footer') }}
		</NcCheckboxRadioSwitch>

		<div class="disclaimer_group">
			<div class="grow_title">
				<span>{{ t('polls', 'Additional email disclaimer') }}</span>
				<LanguageMarkdownIcon />
			</div>
			<NcCheckboxRadioSwitch :checked.sync="preview" type="switch">
				{{ t('polls', 'Preview') }}
			</NcCheckboxRadioSwitch>
		</div>
		<textarea v-show="!preview" v-model="disclaimer" @change="saveSettings()" />
		<!-- eslint-disable-next-line vue/no-v-html -->
		<div v-show="preview" class="polls-markdown" v-html="markedDisclaimer" />
	</div>
</template>

<script>

import { mapState } from 'vuex'
import { NcCheckboxRadioSwitch } from '@nextcloud/vue'
import { marked } from 'marked'
import { gfmHeadingId } from 'marked-gfm-heading-id'
import DOMPurify from 'dompurify'
import { writeValue } from '../../../mixins/adminSettingsMixin.js'
import LanguageMarkdownIcon from 'vue-material-design-icons/LanguageMarkdown.vue'

const markedPrefix = {
	prefix: 'disclaimer-',
}

export default {
	name: 'AdminEmail',

	components: {
		NcCheckboxRadioSwitch,
		LanguageMarkdownIcon,
	},

	mixins: [writeValue],

	data() {
		return {
			preview: false,
		}
	},

	computed: {
		...mapState({
			appSettings: (state) => state.appSettings,
		}),

		markedDisclaimer() {
			marked.use(gfmHeadingId(markedPrefix))
			return DOMPurify.sanitize(marked.parse(this.appSettings.disclaimer))
		},

		// Add bindings
		legalTermsInEmail: {
			get() {
				return !!this.appSettings.legalTermsInEmail
			},
			set(value) {
				this.writeValue({ legalTermsInEmail: !!value })
			},
		},
		disclaimer: {
			get() {
				return this.appSettings.disclaimer
			},
			set(value) {
				this.$store.commit('appSettings/set', { disclaimer: value })
			},
		},
	},

	methods: {
		saveSettings() {
			this.$store.dispatch('appSettings/write')
		},
	},
}
</script>

<style lang="scss">
	.disclaimer_group {
		display: flex;
		align-items: center;

		.grow_title {
			display: flex;
			flex-grow: 1;
			margin-right: 12px;

			.material-design-icon {
				margin-left: 4px;
			}
		}
	}
</style>
