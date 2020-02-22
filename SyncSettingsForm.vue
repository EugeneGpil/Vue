<template>
	<div v-if="amocrmAccountInfo && moyskladAccountInfo">

		<div class="form-group col-lg-6 mt-3">
			<label>Перенаправления</label>
			<input
				class="form-control"
				v-model="settings.redirects"
				placeholder="URI через запятую"
			>
		</div>

		<div class="col-lg-6 mb-3" v-if="slug">
			<div class="input-group">
				<div class="input-group-prepend">
					<label class="input-group-text">Вебхук</label>
				</div>
				<input readonly type="text" class="form-control" :value="webhookLink">
			</div>

			<button class="btn btn-primary mt-3" @click.prevent="setWebhooks">
				Установить вебхуки
			</button>

			<button class="btn btn-danger mt-3 ml-2" @click.prevent="removeWebhooks">
				Удалить вебхуки
			</button>
		</div>

		<div class="col-lg-6 mb-3" v-if="slug">
			<div class="input-group">
				<div class="input-group-prepend">
					<label class="input-group-text">Cпособо синхронизации</label>
				</div>
				<select class="custom-select" required v-model="settings.sync_type">
					<option value="externalCode">Внешний код</option>
					<option value="name">Номер заказа</option>
				</select>
			</div>
		</div>

		<div class="col-lg-6">
			<h3>Синхронизация статусов</h3>
			
			<template v-for="(setting, index) in settings['status_settings']">
				<status-setting
					:setting="setting"
					:amocrmAccountInfo="amocrmAccountInfo"
					:moyskladAccountInfo="moyskladAccountInfo"
					:key="index"
					:number="index + 1"
				/>
			</template>
			
			<div class="d-flex justify-content-end mt-3 mr-1">
		    	<button
					class="btn btn-labeled mb-2 col-1 elevation-2"
					:class="{ 'btn-green' : !isStatusSettingsLimitExceeded(), 
						'btn-dark' : isStatusSettingsLimitExceeded() }"
					@click.prevent="addStatusSetting"
				>
					<span class="btn-label"><i class="fa fa-plus"></i></span>
		    		Добавить
		    	</button>
		    </div>
	    </div>
	 
	    <div class="col-lg-6">
			<h3>Синхронизация полей</h3>

			<div class="card">
				<div class="card-header">
					<strong>Синхронизовать номенклатуру</strong>
					<div class="float-right">
						<label class="switch">
							<input type="checkbox" 
								:checked="settings['nomenclature_enabled']"
								@change="toggleNomenclatureEnabled(settings['nomenclature_enabled'])"
							>
							<span></span>
						</label>
					</div>
				</div>
			</div>

			<template v-for="(setting, index) in settings['fields_settings']">
				<fields-setting
					:setting="setting"
					:amocrmAccountInfo="amocrmAccountInfo"
					:moyskladAccountInfo="moyskladAccountInfo"
					:onlyForReadSettingsList="onlyForReadSettingsList"
					:syncType="settings['sync_type']"
					:key="index"
					:number="index + 1"
				/>
			</template>
			
			<div class="d-flex justify-content-end mt-3 mr-1">
		    	<button
					class="btn btn-labeled mb-2 col-1 elevation-2"
					:class="{ 'btn-green' : !isFieldsSettingsLimitExceeded(), 
						'btn-dark' : isFieldsSettingsLimitExceeded() }"
					@click.prevent="addFieldsSetting"
				>
					<span class="btn-label"><i class="fa fa-plus"></i></span>
		    		Добавить
		    	</button>
		    </div>
	    </div>

	    <div class="col-lg-6">
	    	<button 
				class="btn btn-primary"
				:class="{ 'btn-dark' : !isSettingsValid()
					|| isFieldsSettingsLimitExceededForSave() || isStatusSettingsLimitExceededForSave() }"
				@click.prevent="saveSettings">
	    		Сохранить
	    	</button>
	    </div>
	</div>
	<div v-else><div class="col-lg-6"><p>Загрузка...</p></div></div>
</template>

<script>
    import StatusSetting from "./StatusSetting.vue";
    import FieldsSetting from "./FieldsSetting.vue";
    
    export default {
		name: "sync-settings-form",

		data: () => {
			return {
				onlyForReadSettingsList: {
					"amocrm": [
						"lead-lead_link",
						"company-company_link",
						"contact-contact_link"
					],
					"moysklad": [
						"customerorder-customerorder_sum",
						"customerorder-customerorder_link",
						"customerorder-customerorder_external_code",
						"counterparty-counterparty_link"
					]
				},
				statusLimitExceededMessage: "Превышен лимит правил синхронизации статусов",
				fieldsLimitExceededMessage: "Превышен лимит правил синхронизации полей."
			}
		},
		
		mounted() {
	    	this.$store.dispatch("getAmocrmAccountInfo");
	    	this.$store.dispatch("getMoyskladAccountInfo");
	    	this.$store.dispatch("getMoyskladSettings", this.$store.getters.currentUser.id);
		},

		computed: {
			amocrmAccountInfo() {
				return this.$store.getters.amocrmAccountInfo;
			},

			settings() {
				return this.$store.getters.moyskladSettings;
			},

			slug() {
				return this.$store.getters.moyskladSettingsSlug;
			},

			moyskladAccountInfo() {
				return this.$store.getters.moyskladAccountInfo;
			},

			webhookLink() {
				return `${window.location.origin}/api/moysklad/handle?slug=${this.settings.slug}`;
			},
	    },

	    methods: {
			toggleNomenclatureEnabled(isNomenclatureEnabled) {
				this.$store.commit("toggleNomenclatureEnabled", isNomenclatureEnabled);
			},

	    	addStatusSetting() {
				if (this.isStatusSettingsLimitExceeded()) {
					this.$store.dispatch("infoPopup", this.statusLimitExceededMessage);
				} else {
					this.$store.commit("addMoyskladStatusSettings");
				}
	    	},

	    	addFieldsSetting() {
				if (this.isFieldsSettingsLimitExceeded()) {
					this.$store.dispatch("infoPopup", this.fieldsLimitExceededMessage);
				} else {
					this.$store.commit("addMoyskladFieldsSettings");
				}
	    	},

			saveSettings() {
				if (!this.isSettingsValid()) {
					this.$store.dispatch("errorPopup", "Ошибка в одном или нескольких правилах синхронинзации полей.");
					return;
				}

				if (this.isStatusSettingsLimitExceededForSave()) {
					this.$store.dispatch("infoPopup", this.statusLimitExceededMessage);
					return;
				}

				if (this.isFieldsSettingsLimitExceededForSave()) {
					this.$store.dispatch("infoPopup", this.fieldsLimitExceededMessage);
					return;
				}

				if (this.settings.slug) {
					this.$store.dispatch("updateMoyskladSettings", this.settings);
				} else {
					this.$store.dispatch("saveMoyskladSettings");
				}
			},
			
			setWebhooks() {
				this.$store.dispatch("setWebhooks");
			},

			removeWebhooks() {
				this.$store.dispatch("swatCheck", "Вы уверены? Удаление вебхуков приведет к отключению модуля.")
					.then((result) => {
						if (result.value) {
							this.$store.dispatch("removeMoysklad24Webhooks");
						}
					})
			},

			isSettingsValid() {
				for (let i = 0; i < this.settings.fields_settings.length; i++) {
					if (this.settings.fields_settings[i].direction == "amocrm") {
						if (this.onlyForReadSettingsList.amocrm
							.indexOf(this.settings.fields_settings[i].amocrm_field) != -1) {
								return false;
						}
					} else {
						let amocrmFieldType = this.getAmocrmFieldType(this.settings.fields_settings[i].amocrm_field);
						let moyskladFieldType = this.getMoyskladFieldType(this.settings.fields_settings[i].moysklad_field);
			
						if (this.onlyForReadSettingsList.moysklad
							.indexOf(this.settings.fields_settings[i].moysklad_field) != -1
							|| (this.settings.sync_type == "name"
							&& this.settings.fields_settings[i].moysklad_field == "customerorder-customerorder_name")
							|| (!(amocrmFieldType == 6 || amocrmFieldType == 14) && moyskladFieldType == 'time' )) {
								return false;
						}
					}
				}
				return true;
			},

			isStatusSettingsLimitExceededForSave() {
				if (this.settings.status_settings.length > this.settings.new_status_settings_limit) {
					return true;
				}
				return false;
			},

			isStatusSettingsLimitExceeded() {
				if (this.settings.status_settings.length >= this.settings.new_status_settings_limit) {
					return true;
				}
				return false;
			},

			isFieldsSettingsLimitExceededForSave() {
				if (this.settings.fields_settings.length > this.settings.new_fields_settings_limit) {
					return true;
				}
				return false;
			},

			isFieldsSettingsLimitExceeded() {
				if (this.settings.fields_settings.length >= this.settings.new_fields_settings_limit) {
					return true;
				}
				return false;
			},

			getAmocrmFieldType(amocrmField) {
                if (!amocrmField) {
                    return null;
                }

                const fieldArray = amocrmField.split('-');
                const entity = fieldArray[0] == 'company' ? 'companies' : fieldArray[0] + 's';
                const fieldId = fieldArray[1];

                for (let i = 0; i < this.amocrmAccountInfo.custom_fields[entity].length; i++) {
                    if (this.amocrmAccountInfo.custom_fields[entity][i].id == fieldId) {
                        return this.amocrmAccountInfo.custom_fields[entity][i].field_type;
                    }
                }
                return null;
            },

            getMoyskladFieldType(moyskladField) {
                if (!moyskladField) {
                    return null;
                }

                const positionOfDash = moyskladField.indexOf('-');
                const entity = moyskladField.substr(0, positionOfDash) + '_meta';
                const fieldId = moyskladField.substr(positionOfDash + 1);

                for (let i = 0; i < this.moyskladAccountInfo[entity].attributes.length; i++) {
                    if (this.moyskladAccountInfo[entity].attributes[i].id == fieldId) {
                        return this.moyskladAccountInfo[entity].attributes[i].type;
                    }
                }

                return null;
            }
	    },

	    components: {
	    	StatusSetting,
			FieldsSetting
	    }
	}
</script>

<style scoped>
	.switch {
		margin-bottom: 0;
	}

	.btn.btn-labeled.btn-dark, .btn-dark, .btn.btn-primary.btn-dark:active{
		background-color: #aaabaf;
		border-color: #aaabaf;
	}
</style>