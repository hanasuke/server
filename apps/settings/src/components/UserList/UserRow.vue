<!--
  - @copyright Copyright (c) 2018 John Molakvoæ <skjnldsv@protonmail.com>
  - @copyright Copyright (c) 2019 Gary Kim <gary@garykim.dev>
  -
  - @author John Molakvoæ <skjnldsv@protonmail.com>
  - @author Gary Kim <gary@garykim.dev>
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
  - along with this program. If not, see <http://www.gnu.org/licenses/>.
  -
  -->

<template>
	<!-- Obfuscated user: Logged in user does not have permissions to see all of the data -->
	<div v-if="Object.keys(user).length ===1" :data-id="user.id" class="row">
		<div :class="{'icon-loading-small': loading.delete || loading.disable || loading.wipe}"
			class="avatar">
			<img v-if="!loading.delete && !loading.disable && !loading.wipe"
				:src="generateAvatar(user.id, isDarkTheme)"
				alt=""
				height="32"
				width="32">
		</div>
		<div class="name">
			{{ user.id }}
		</div>
		<div class="obfuscated">
			{{ t('settings','You do not have permissions to see the details of this user') }}
		</div>
	</div>

	<!-- User full data -->
	<UserRowSimple v-else-if="!editing"
		:editing.sync="editing"
		:feedback-message="feedbackMessage"
		:groups="groups"
		:languages="languages"
		:loading="loading"
		:opened-menu.sync="openedMenu"
		:settings="settings"
		:show-config="showConfig"
		:sub-admins-groups="subAdminsGroups"
		:user-actions="userActions"
		:user="user"
		:is-dark-theme="isDarkTheme"
		:class="{'row--menu-opened': openedMenu}" />
	<tr v-else
		:class="{
			'disabled': loading.delete || loading.disable,
			'row--menu-opened': openedMenu
		}"
		:data-id="user.id"
		class="row row--editable">
		<td :class="{'icon-loading-small': loading.delete || loading.disable || loading.wipe}"
			class="avatar">
			<img v-if="!loading.delete && !loading.disable && !loading.wipe"
				:src="generateAvatar(user.id, isDarkTheme)"
				alt=""
				height="32"
				width="32">
		</td>
		<!-- dirty hack to ellipsis on two lines -->
		<td v-if="user.backendCapabilities.setDisplayName" class="displayName">
			<form :class="{'icon-loading-small': loading.displayName}"
				class="displayName"
				@submit.prevent="updateDisplayName">
				<label class="hidden-visually" :for="'displayName'+user.id+rand">{{ t('settings', 'Edit display name') }}</label>
				<input :id="'displayName'+user.id+rand"
					ref="displayName"
					:disabled="loading.displayName||loading.all"
					:value="user.displayname"
					autocapitalize="off"
					autocomplete="off"
					autocorrect="off"
					spellcheck="false"
					type="text">
				<input class="icon-confirm"
					type="submit"
					value="">
			</form>
		</td>
		<td v-else class="name">
			{{ user.id }}
			<div class="displayName subtitle">
				<div :title="user.displayname.length > 20 ? user.displayname : ''" class="cellText">
					{{ user.displayname }}
				</div>
			</div>
		</td>
		<td v-if="settings.canChangePassword && user.backendCapabilities.setPassword">
			<form :class="{'icon-loading-small': loading.password}"
				class="password"
				@submit.prevent="updatePassword">
				<label class="hidden-visually" :for="'password'+user.id+rand">{{ t('settings', 'Add new password') }}</label>
				<input :id="'password'+user.id+rand"
					ref="password"
					:disabled="loading.password || loading.all"
					:minlength="minPasswordLength"
					maxlength="469"
					:placeholder="t('settings', 'Add new password')"
					autocapitalize="off"
					autocomplete="new-password"
					autocorrect="off"
					required
					spellcheck="false"
					type="password"
					value="">
				<input class="icon-confirm" type="submit" value="">
			</form>
		</td>
		<td v-else />
		<td>
			<form :class="{'icon-loading-small': loading.mailAddress}"
				class="mailAddress"
				@submit.prevent="updateEmail">
				<label class="hidden-visually" :for="'mailAddress'+user.id+rand">{{ t('settings', 'Add new email address') }}</label>
				<input :id="'mailAddress'+user.id+rand"
					ref="mailAddress"
					:disabled="loading.mailAddress||loading.all"
					:placeholder="t('settings', 'Add new email address')"
					:value="user.email"
					autocapitalize="off"
					autocomplete="new-password"
					autocorrect="off"
					spellcheck="false"
					type="email">
				<input class="icon-confirm" type="submit" value="">
			</form>
		</td>
		<td :class="{'icon-loading-small': loading.groups}" class="groups">
			<label class="hidden-visually" :for="'groups'+user.id+rand">{{ t('settings', 'Add user to group') }}</label>
			<NcMultiselect :id="'groups'+user.id+rand"
				:close-on-select="false"
				:disabled="loading.groups||loading.all"
				:limit="2"
				:multiple="true"
				:options="availableGroups"
				:placeholder="t('settings', 'Add user to group')"
				:tag-width="60"
				:taggable="settings.isAdmin"
				:value="userGroups"
				class="multiselect-vue"
				label="name"
				tag-placeholder="create"
				track-by="id"
				@remove="removeUserGroup"
				@select="addUserGroup"
				@tag="createGroup">
				<span slot="noResult">{{ t('settings', 'No results') }}</span>
			</NcMultiselect>
		</td>
		<td v-if="subAdminsGroups.length>0 && settings.isAdmin"
			:class="{'icon-loading-small': loading.subadmins}"
			class="subadmins">
			<label class="hidden-visually" :for="'subadmins'+user.id+rand">{{ t('settings', 'Set user as admin for') }}</label>
			<NcMultiselect :id="'subadmins'+user.id+rand"
				:close-on-select="false"
				:disabled="loading.subadmins||loading.all"
				:limit="2"
				:multiple="true"
				:options="subAdminsGroups"
				:placeholder="t('settings', 'Set user as admin for')"
				:tag-width="60"
				:value="userSubAdminsGroups"
				class="multiselect-vue"
				label="name"
				track-by="id"
				@remove="removeUserSubAdmin"
				@select="addUserSubAdmin">
				<span slot="noResult">{{ t('settings', 'No results') }}</span>
			</NcMultiselect>
		</td>
		<td :title="usedSpace"
			:class="{'icon-loading-small': loading.quota}"
			class="quota">
			<label class="hidden-visually" :for="'quota'+user.id+rand">{{ t('settings', 'Select user quota') }}</label>
			<NcMultiselect :id="'quota'+user.id+rand"
				:allow-empty="false"
				:disabled="loading.quota||loading.all"
				:options="quotaOptions"
				:placeholder="t('settings', 'Select user quota')"
				:taggable="true"
				:value="userQuota"
				class="multiselect-vue"
				label="label"
				tag-placeholder="create"
				track-by="id"
				@input="setUserQuota"
				@tag="validateQuota" />
		</td>
		<td v-if="showConfig.showLanguages"
			:class="{'icon-loading-small': loading.languages}"
			class="languages">
			<label class="hidden-visually" :for="'language'+user.id+rand">{{ t('settings', 'Set the language') }}</label>
			<NcMultiselect :id="'language'+user.id+rand"
				:allow-empty="false"
				:disabled="loading.languages||loading.all"
				:options="languages"
				:placeholder="t('settings', 'No language set')"
				:value="userLanguage"
				class="multiselect-vue"
				group-label="label"
				group-values="languages"
				label="name"
				track-by="code"
				@input="setUserLanguage" />
		</td>
		<td :class="{'icon-loading-small': loading.manager}" class="managers">
			<NcMultiselect ref="manager"
				v-model="currentManager"
				:close-on-select="true"
				:user-select="true"
				:options="possibleManagers"
				:placeholder="t('settings', 'Select manager')"
				class="multiselect-vue"
				label="displayname"
				track-by="id"
				@search-change="searchUserManager"
				@remove="updateUserManager"
				@select="updateUserManager">
				<span slot="noResult">{{ t('settings', 'No results') }}</span>
			</NcMultiselect>
		</td>

		<!-- don't show this on edit mode -->
		<td v-if="showConfig.showStoragePath || showConfig.showUserBackend"
			class="storageLocation" />
		<td v-if="showConfig.showLastLogin" />

		<td class="userActions">
			<div v-if="!loading.all"
				class="toggleUserActions">
				<NcActions>
					<NcActionButton icon="icon-checkmark"
						:title="t('settings', 'Done')"
						:aria-label="t('settings', 'Done')"
						@click="editing = false" />
				</NcActions>
				<div v-click-outside="hideMenu" class="userPopoverMenuWrapper">
					<button class="icon-more"
						:aria-expanded="openedMenu"
						:aria-label="t('settings', 'Toggle user actions menu')"
						@click.prevent="toggleMenu" />
					<div :class="{ 'open': openedMenu }" class="popovermenu">
						<NcPopoverMenu :menu="userActions" />
					</div>
				</div>
			</div>
			<div :style="{opacity: feedbackMessage !== '' ? 1 : 0}"
				class="feedback">
				<div class="icon-checkmark" />
				{{ feedbackMessage }}
			</div>
		</td>
	</tr>
</template>

<script>
import ClickOutside from 'vue-click-outside'

import {
	NcPopoverMenu,
	NcMultiselect,
	NcActions,
	NcActionButton,
} from '@nextcloud/vue'
import UserRowSimple from './UserRowSimple.vue'
import UserRowMixin from '../../mixins/UserRowMixin.js'

export default {
	name: 'UserRow',
	components: {
		UserRowSimple,
		NcPopoverMenu,
		NcActions,
		NcActionButton,
		NcMultiselect,
	},
	directives: {
		ClickOutside,
	},
	mixins: [UserRowMixin],
	props: {
		users: {
			type: Array,
			required: true,
		},
		user: {
			type: Object,
			required: true,
		},
		settings: {
			type: Object,
			default: () => ({}),
		},
		groups: {
			type: Array,
			default: () => [],
		},
		subAdminsGroups: {
			type: Array,
			default: () => [],
		},
		quotaOptions: {
			type: Array,
			default: () => [],
		},
		showConfig: {
			type: Object,
			default: () => ({}),
		},
		languages: {
			type: Array,
			required: true,
		},
		externalActions: {
			type: Array,
			default: () => [],
		},
		isDarkTheme: {
			type: Boolean,
			required: true,
		},
	},
	data() {
		return {
			rand: parseInt(Math.random() * 1000),
			openedMenu: false,
			feedbackMessage: '',
			possibleManagers: [],
			currentManager: '',
			editing: false,
			loading: {
				all: false,
				displayName: false,
				password: false,
				mailAddress: false,
				groups: false,
				subadmins: false,
				quota: false,
				delete: false,
				disable: false,
				languages: false,
				wipe: false,
				manager: false,
			},
		}
	},
	computed: {

		/* USER POPOVERMENU ACTIONS */
		userActions() {
			const actions = [
				{
					icon: 'icon-delete',
					text: t('settings', 'Delete user'),
					action: this.deleteUser,
				},
				{
					icon: 'icon-delete',
					text: t('settings', 'Wipe all devices'),
					action: this.wipeUserDevices,
				},
				{
					icon: this.user.enabled ? 'icon-close' : 'icon-add',
					text: this.user.enabled ? t('settings', 'Disable user') : t('settings', 'Enable user'),
					action: this.enableDisableUser,
				},
			]
			if (this.user.email !== null && this.user.email !== '') {
				actions.push({
					icon: 'icon-mail',
					text: t('settings', 'Resend welcome email'),
					action: this.sendWelcomeMail,
				})
			}
			return actions.concat(this.externalActions)
		},
	},
	async beforeMount() {
		await this.searchUserManager()
		if (this.user.manager) {
			await this.initManager(this.user.manager)
		}
	},

	methods: {
		/* MENU HANDLING */
		toggleMenu() {
			this.openedMenu = !this.openedMenu
		},
		hideMenu() {
			this.openedMenu = false
		},

		wipeUserDevices() {
			const userid = this.user.id
			OC.dialogs.confirmDestructive(
				t('settings', 'In case of lost device or exiting the organization, this can remotely wipe the Nextcloud data from all devices associated with {userid}. Only works if the devices are connected to the internet.', { userid }),
				t('settings', 'Remote wipe of devices'),
				{
					type: OC.dialogs.YES_NO_BUTTONS,
					confirm: t('settings', 'Wipe {userid}\'s devices', { userid }),
					confirmClasses: 'error',
					cancel: t('settings', 'Cancel'),
				},
				(result) => {
					if (result) {
						this.loading.wipe = true
						this.loading.all = true
						this.$store.dispatch('wipeUserDevices', userid)
							.then(() => {
								this.loading.wipe = false
								this.loading.all = false
							})
					}
				},
				true
			)
		},

		filterManagers(managers) {
			return managers.filter((manager) => manager.id !== this.user.id)
		},
		async initManager(userId) {
			await this.$store.dispatch('getUser', userId).then(response => {
				this.currentManager = response?.data.ocs.data
			})
		},
		async searchUserManager(query) {
			await this.$store.dispatch('searchUsers', { offset: 0, limit: 10, search: query }).then(response => {
				const users = response?.data ? this.filterManagers(Object.values(response?.data.ocs.data.users)) : []
				if (users.length > 0) {
					this.possibleManagers = users
				}
			})
		},

		updateUserManager(manager) {
			this.loading.manager = true
			this.$store.dispatch('setUserData', {
				userid: this.user.id,
				key: 'manager',
				value: this.currentManager ? this.currentManager.id : '',
			}).then(() => {
				this.loading.manager = false
			})
		},

		deleteUser() {
			const userid = this.user.id
			OC.dialogs.confirmDestructive(
				t('settings', 'Fully delete {userid}\'s account including all their personal files, app data, etc.', { userid }),
				t('settings', 'Account deletion'),
				{
					type: OC.dialogs.YES_NO_BUTTONS,
					confirm: t('settings', 'Delete {userid}\'s account', { userid }),
					confirmClasses: 'error',
					cancel: t('settings', 'Cancel'),
				},
				(result) => {
					if (result) {
						this.loading.delete = true
						this.loading.all = true
						return this.$store.dispatch('deleteUser', userid)
							.then(() => {
								this.loading.delete = false
								this.loading.all = false
							})
					}
				},
				true
			)
		},

		enableDisableUser() {
			this.loading.delete = true
			this.loading.all = true
			const userid = this.user.id
			const enabled = !this.user.enabled
			return this.$store.dispatch('enableDisableUser', {
				userid,
				enabled,
			})
				.then(() => {
					this.loading.delete = false
					this.loading.all = false
				})
		},

		/**
		 * Set user displayName
		 *
		 * @param {string} displayName The display name
		 */
		updateDisplayName() {
			const displayName = this.$refs.displayName.value
			this.loading.displayName = true
			this.$store.dispatch('setUserData', {
				userid: this.user.id,
				key: 'displayname',
				value: displayName,
			}).then(() => {
				this.loading.displayName = false
				this.$refs.displayName.value = displayName
			})
		},

		/**
		 * Set user password
		 *
		 * @param {string} password The email address
		 */
		updatePassword() {
			const password = this.$refs.password.value
			this.loading.password = true
			this.$store.dispatch('setUserData', {
				userid: this.user.id,
				key: 'password',
				value: password,
			}).then(() => {
				this.loading.password = false
				this.$refs.password.value = '' // empty & show placeholder
			})
		},

		/**
		 * Set user mailAddress
		 *
		 * @param {string} mailAddress The email address
		 */
		updateEmail() {
			const mailAddress = this.$refs.mailAddress.value
			this.loading.mailAddress = true
			this.$store.dispatch('setUserData', {
				userid: this.user.id,
				key: 'email',
				value: mailAddress,
			}).then(() => {
				this.loading.mailAddress = false
				this.$refs.mailAddress.value = mailAddress
			})
		},

		/**
		 * Create a new group and add user to it
		 *
		 * @param {string} gid Group id
		 */
		async createGroup(gid) {
			this.loading = { groups: true, subadmins: true }
			try {
				await this.$store.dispatch('addGroup', gid)
				const userid = this.user.id
				await this.$store.dispatch('addUserGroup', { userid, gid })
			} catch (error) {
				console.error(error)
			} finally {
				this.loading = { groups: false, subadmins: false }
			}
			return this.$store.getters.getGroups[this.groups.length]
		},

		/**
		 * Add user to group
		 *
		 * @param {object} group Group object
		 */
		async addUserGroup(group) {
			if (group.canAdd === false) {
				return false
			}
			this.loading.groups = true
			const userid = this.user.id
			const gid = group.id
			try {
				await this.$store.dispatch('addUserGroup', { userid, gid })
			} catch (error) {
				console.error(error)
			} finally {
				this.loading.groups = false
			}
		},

		/**
		 * Remove user from group
		 *
		 * @param {object} group Group object
		 */
		async removeUserGroup(group) {
			if (group.canRemove === false) {
				return false
			}

			this.loading.groups = true
			const userid = this.user.id
			const gid = group.id

			try {
				await this.$store.dispatch('removeUserGroup', {
					userid,
					gid,
				})
				this.loading.groups = false
				// remove user from current list if current list is the removed group
				if (this.$route.params.selectedGroup === gid) {
					this.$store.commit('deleteUser', userid)
				}
			} catch {
				this.loading.groups = false
			}
		},

		/**
		 * Add user to group
		 *
		 * @param {object} group Group object
		 */
		async addUserSubAdmin(group) {
			this.loading.subadmins = true
			const userid = this.user.id
			const gid = group.id

			try {
				await this.$store.dispatch('addUserSubAdmin', {
					userid,
					gid,
				})
				this.loading.subadmins = false
			} catch (error) {
				console.error(error)
			}
		},

		/**
		 * Remove user from group
		 *
		 * @param {object} group Group object
		 */
		async removeUserSubAdmin(group) {
			this.loading.subadmins = true
			const userid = this.user.id
			const gid = group.id

			try {
				await this.$store.dispatch('removeUserSubAdmin', {
					userid,
					gid,
				})
			} catch (error) {
				console.error(error)
			} finally {
				this.loading.subadmins = false
			}
		},

		/**
		 * Dispatch quota set request
		 *
		 * @param {string | object} quota Quota in readable format '5 GB' or Object {id: '5 GB', label: '5GB'}
		 * @return {string}
		 */
		async setUserQuota(quota = 'none') {
			this.loading.quota = true
			// ensure we only send the preset id
			quota = quota.id ? quota.id : quota

			try {
				await this.$store.dispatch('setUserData', {
					userid: this.user.id,
					key: 'quota',
					value: quota,
				})
			} catch (error) {
				console.error(error)
			} finally {
				this.loading.quota = false
			}
			return quota
		},

		/**
		 * Validate quota string to make sure it's a valid human file size
		 *
		 * @param {string} quota Quota in readable format '5 GB'
		 * @return {Promise|boolean}
		 */
		validateQuota(quota) {
			// only used for new presets sent through @Tag
			const validQuota = OC.Util.computerFileSize(quota)
			if (validQuota !== null && validQuota >= 0) {
				// unify format output
				return this.setUserQuota(OC.Util.humanFileSize(OC.Util.computerFileSize(quota)))
			}
			// if no valid do not change
			return false
		},

		/**
		 * Dispatch language set request
		 *
		 * @param {object} lang language object {code:'en', name:'English'}
		 * @return {object}
		 */
		async setUserLanguage(lang) {
			this.loading.languages = true
			// ensure we only send the preset id
			try {
				await this.$store.dispatch('setUserData', {
					userid: this.user.id,
					key: 'language',
					value: lang.code,
				})
			} catch (error) {
				console.error(error)
			} finally {
				this.loading.languages = false
			}
			return lang
		},

		/**
		 * Dispatch new welcome mail request
		 */
		sendWelcomeMail() {
			this.loading.all = true
			this.$store.dispatch('sendWelcomeMail', this.user.id)
				.then(success => {
					if (success) {
						// Show feedback to indicate the success
						this.feedbackMessage = t('setting', 'Welcome mail sent!')
						setTimeout(() => {
							this.feedbackMessage = ''
						}, 2000)
					}
					this.loading.all = false
				})
		},

	},
}
</script>
<style scoped lang="scss">
	// Force menu to be above other rows
	.row--menu-opened {
		z-index: 1 !important;
	}
	.row::v-deep .multiselect__single {
		z-index: auto !important;
	}
	.displayName input,
	.password input,
	.mailAddress input {
	  width: 100%;
	  height: 44px!important;
		border: 2px solid var(--color-border-dark);
	}
</style>
