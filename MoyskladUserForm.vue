<template>
	<div class="card card-default col-xl-5 mr-3 ml-3">
	    <div class="card-header">
	    	<span>Настройки пользователя Moysklad</span>
	    </div>
	    <div class="card-body">
	        <form role="form" @submit.prevent="updateMoyskladUser">
		        <div class="form-group">
		            <label>Moysklad логин</label>
		            <input name="moysklad_user" type="text" class="form-control" v-model="moyskladUser.login" required>
		        </div>
		        <div class="form-group">
		            <label>Moysklad пароль</label>
		            <input
		            	type="password"
		            	class="form-control"
		            	v-model="moyskladUser.password"
		            	required
		            	placeholder="Введите новый пароль"
		            >
		        </div>
		        <button class="btn btn-sm btn-secondary" type="submit">Сохранить</button>
	        </form>
	    </div>
	</div>
</template>

<script>
	export default {
		name: "moysklad-user-form",
		computed: {
			moyskladUser() {
				return this.$store.getters.moyskladUser;
			}
		},
		methods: {
	        updateMoyskladUser() {
	        	this.$store.dispatch("updateMoyskladUser", this.moyskladUser);
	        }
		},
		mounted() {
			this.$store.dispatch("getMoyskladUser");
		}
	}
</script>