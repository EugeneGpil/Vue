<template>
    <div class="card">
        <summary
            class="card-header"
            data-tool="card-collapse"
            data-toggle="tooltip"
            :class="{ invalid : isFieldInvalid(setting.direction) }"
        >
            <strong>{{ number + " " + getPreview() }}</strong>
            <div class="float-right">
                <em
                    class="fa fa-plus open-setting"
                    :class="{ invalid : isFieldInvalid(setting.direction),
                        'text-dark' : !isFieldInvalid(setting.direction) }"
                ></em>
            </div>
        </summary>
        <div class="card-wrapper collapse p-3">
            <div class="form-group input-group ">
                <div class="input-group-prepend">
                    <label class="input-group-text" :class="{ invalid : isFieldInvalid('amocrm')}">
                        Поле amocrm
                    </label>
                </div>
                        
                <select
                    v-if="amocrmAccountInfo.custom_fields"
                    v-model="setting.amocrm_field"
                    class="custom-select"
                    :class="{ invalid : isFieldInvalid('amocrm')}"
                    required
                >
                    <optgroup label="Сделка" class="select-option">
                        <option
                           v-for="field in amocrmAccountInfo.custom_fields.leads"
                            :value="`${field.entity}-${field.id}`"
                            :key="field.id"
                        >{{ `${field.prefix} ${field.name}` }}</option>
                    </optgroup>

                    <optgroup label="Компания сделки" class="select-option">
                        <option
                           v-for="field in amocrmAccountInfo.custom_fields.companies"
                            :value="`${field.entity}-${field.id}`"
                            :key="field.id"
                        >{{ `${field.prefix} ${field.name}` }}</option>
                    </optgroup>

                    <optgroup label="Контакт сделки" class="select-option">
                        <option
                           v-for="field in amocrmAccountInfo.custom_fields.contacts"
                            :value="`${field.entity}-${field.id}`"
                            :key="field.id"
                        >{{ `${field.prefix} ${field.name}` }}</option>
                    </optgroup>
                </select>
            </div>

            <div class="form-group input-group ">
                <div class="input-group-prepend">
                    <label class="input-group-text" :class="{ invalid : isFieldInvalid('moysklad') }">
                        Поле МойСклад
                    </label>
                </div>
                <select 
                    v-model="setting.moysklad_field"
                    class="custom-select" 
                    :class="{ invalid : isFieldInvalid('moysklad') }"
                    required
                >
                    <optgroup label="Заказ покупателя" class="select-option">
                        <option
                            v-for="field in moyskladAccountInfo.customerorder_meta.attributes"
                            :value="`${field.entity}-${field.id}`"
                            :key="field.id"
                        >{{ `${field.prefix} ${field.name}` }}</option>
                    </optgroup>

                    <optgroup label="Контрагент заказа" class="select-option">
                        <option
                            v-for="field in moyskladAccountInfo.counterparty_meta.attributes"
                            :value="`${field.entity}-${field.id}`"
                            :key="field.id"
                        >{{ `${field.prefix} ${field.name}` }}</option>
                    </optgroup>
                </select>
            </div>

            <div class="form-group input-group ">
                <div class="input-group-prepend">
                    <label class="input-group-text">Направление</label>
                </div>
                <select v-model="setting.direction" class="custom-select" required>
                    <option value="amocrm">Из МоегоСклада в amocrm</option>
                    <option value="moysklad">Из amocrm в МойСклад</option>
                </select>
            </div>

            <div class="d-flex justify-content-end mt-3">
		    	<button class="btn btn-danger mb-2 col-1 elevation-2" @click.prevent="removeSetting(setting)">Удалить</button>
		    </div>
	    </div>
    </div>
</template>

<script>
	export default {
		name: "moysklad-field-setting",
        props: [
            "setting",
            "amocrmAccountInfo",
            "moyskladAccountInfo",
            "onlyForReadSettingsList",
            "syncType",
            "number"
        ],
        
		methods: {
			removeSetting(setting) {
				this.$store.dispatch("swatCheck")
		            .then((result) => {
		                if (result.value) {
        					this.$store.commit("removeMoyskladSettings", setting);
		                }
		            });
            },

            // TODO rewrite
            getPreview() {
                let str = '';

                _.each(this.amocrmAccountInfo.custom_fields, (type, ind) => {
                    _.each(type, (cf, ind) => {
                        if (`${cf.entity}-${cf.id}` == this.setting.amocrm_field) {
                            str += `"${cf.prefix} ${cf.name}"`;
                        }
                    });
                });

                str += "amoCRM";

                str += this.setting.direction == "amocrm" ? "  ⇦  " : "  ⇨  ";

                str += "МойСклад";

                _.each(this.moyskladAccountInfo.customerorder_meta.attributes, (el, ind) => {
                    if (`${el.entity}-${el.id}` == this.setting.moysklad_field) {
                        str += `"${el.prefix} ${el.name}"`
                    }
                });

                _.each(this.moyskladAccountInfo.counterparty_meta.attributes, (el, ind) => {
                    if (`${el.entity}-${el.id}` == this.setting.moysklad_field) {
                        str += `"${el.prefix} ${el.name}"`
                    }
                });

				return str;
            },

            isFieldInvalid(field) {

                if (field == "amocrm" 
                    && this.setting.direction == "amocrm"
                    && this.onlyForReadSettingsList.amocrm.indexOf(this.setting.amocrm_field) != -1) {
                        return true;
                    }

                const amocrmFieldType = this.getAmocrmFieldType(this.setting.amocrm_field);
                const moyskladFieldType = this.getMoyskladFieldType(this.setting.moysklad_field);
                
                if (field == "moysklad"
                    && this.setting.direction == "moysklad"
                    && (this.onlyForReadSettingsList.moysklad.indexOf(this.setting.moysklad_field) != -1
                    || (this.setting.moysklad_field == "customerorder-customerorder_name" 
                        && this.syncType == "name")
                    || (!(amocrmFieldType == 6 || amocrmFieldType == 14) && moyskladFieldType == 'time'))) {
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

                for (let i = 0; i < this.$props.amocrmAccountInfo.custom_fields[entity].length; i++) {
                    if (this.$props.amocrmAccountInfo.custom_fields[entity][i].id == fieldId) {
                        return this.$props.amocrmAccountInfo.custom_fields[entity][i].field_type;
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

                for (let i = 0; i < this.$props.moyskladAccountInfo[entity].attributes.length; i++) {
                    if (this.$props.moyskladAccountInfo[entity].attributes[i].id == fieldId) {
                        return this.$props.moyskladAccountInfo[entity].attributes[i].type;
                    }
                }

                return null;
            }
        }
	}
</script>

<style scoped>
	.input-group-prepend label {
		width: 170px;
		justify-content: center;
	}
</style>