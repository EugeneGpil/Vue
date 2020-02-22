<template>
    <div class="card">
        <summary class="card-header" data-tool="card-collapse" data-toggle="tooltip">
            <strong>{{ number + " " + getPreview() }}</strong>
            <div class="float-right">
                <em class="fa fa-plus text-dark"></em>
            </div>
        </summary>
        <div class="card-wrapper collapse p-3">

            <div class="form-group input-group ">
                <div class="input-group-prepend">
                    <label class="input-group-text">Тип</label>
                </div>
                <select v-model="setting.type" class="custom-select" required>
                    <option value="sync">Синхронизировать статус</option>
                    <option value="create">Создать сущность</option>
                    <option value="invoiceout">Интеграция со счетами</option>
                    <option value="shipment">Интеграция с отгрузками</option>
                </select>
            </div>

            <div class="form-group input-group ">
                <div class="input-group-prepend">
                    <label class="input-group-text">Статус сделки amocrm</label>
                </div>
                <select v-model="setting.amocrm_status" class="custom-select" required>
                    <optgroup
                        v-for="pipeline in amocrmAccountInfo.pipelines"
                        :label="pipeline.name"
                        :key="pipeline.id"
                    >
                        <option
                            v-for="status in pipeline.statuses"
                            :value="`${status.id}`"
                            :key="status.id"
                        >{{ status.name }}</option>
                    </optgroup>
                </select>
            </div>

            <div class="form-group input-group " v-if="setting.type !== 'invoiceout' && setting.type !== 'shipment'">
                <div class="input-group-prepend">
                    <label class="input-group-text">Статус заказа МойСклад</label>
                </div>
                <select v-model="setting.moysklad_status" class="custom-select" required>
                    <option
                        v-for="state in moyskladAccountInfo.customerorder_meta.states"
                        :value="`${state.id}`"
                        :key="state.id"
                    >{{ state.name }}</option>
                </select>
            </div>

            <div class="form-group input-group " v-if="setting.type !== 'invoiceout' && setting.type !== 'shipment'">
                <div class="input-group-prepend">
                    <label class="input-group-text">Направление</label>
                </div>
                <select v-model="setting.direction" class="custom-select" required>
                    <option value="amocrm">Из МоегоСклада в amocrm</option>
                    <option value="moysklad">Из amocrm в МойСклад</option>
                </select>
            </div>

            <template v-if="setting.type == 'create' && setting.direction == 'moysklad'">
                <div class="form-group input-group">
                    <div class="input-group-prepend">
                        <label class="input-group-text">Контрагент для заказа</label>
                    </div>
                    <select v-model="setting.moysklad_counterparty" class="custom-select" required>
                        <option selected value="default">Привязанный контакт/компания</option>
                        <option value="company">Компания сделки</option>
                        <option value="contact">Главный контакт сделки</option>
                    </select>
                </div>

                <div class="form-group input-group">
                    <div class="input-group-prepend">
                        <label class="input-group-text">Организация для заказа</label>
                    </div>
                    <select v-model="setting.moysklad_organization" class="custom-select" required>
                        <option
                            v-for="organization in moyskladAccountInfo.organizations"
                            :value="`${organization.id}`"
                            :key="organization.id"
                        >{{ organization.name }}</option>
                    </select>
                </div>

                <div class="form-group input-group">
                    <div class="input-group-prepend">
                        <label class="input-group-text">Склад</label>
                    </div>
                    <select v-model="setting.moysklad_store" class="custom-select" required>
                        <option
                            v-for="store in moyskladAccountInfo.stores"
                            :value="`${store.id}`"
                            :key="store.id"
                        >{{ store.name }}</option>
                    </select>
                </div>
            </template>
            

            <template v-if="setting.type == 'create' && setting.direction == 'amocrm'">
                <div class="form-group input-group">
                    <div class="input-group-prepend">
                        <label class="input-group-text">Контрагент заказа</label>
                    </div>
                    <select v-model="setting.amocrm_counterparty" class="custom-select" required>
                        <option selected value="contact">Привязанный контакт</option>
                        <option value="company">Привязанная компания</option>
                    </select>
                </div>
            </template>


            <div class="d-flex justify-content-end mt-3">
		    	<button class="btn btn-danger mb-2 col-1 elevation-2" @click.prevent="removeSetting(setting)">Удалить</button>
		    </div>
        </div>
	</div>
</template>

<script>
	export default {
		name: "moysklad-status-setting",
		props: ["setting", "amocrmAccountInfo", "moyskladAccountInfo", "number"],

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


                _.each(this.amocrmAccountInfo.pipelines, (pipeline, ind) => {
                    _.each(pipeline.statuses, (status, ind) => {
                        if (ind != 142 & ind != 143 & ind == this.setting.amocrm_status) {
                            str += `"${status.name}"`;
                        }
                    });
                });

                str += "amoCRM";

                str += this.setting.direction === "amocrm" ?  ((this.setting.type === 'invoiceout' || this.setting.type === 'shipment') ?
                    "  ⇨  " : "  ⇦  ") : "  ⇨  ";

                str += "МойСклад";

                _.each(this.moyskladAccountInfo.customerorder_meta.states, (el, ind) => {
                    if (el.id == this.setting.moysklad_status) {
                        str += `"${el.name}"`
                    }
                });

                if(this.setting.type === 'invoiceout')
                    str += `"Счет покупателя"`;
                if (this.setting.type === 'shipment')
                    str += `"Отгрузка"`;

				return str;
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
