<template>
	<div>
		<div class="login" v-if="!token">
			<div class="form-group">
				<label>Username</label>
				<input type="text" class="form-control" v-model="loginDTO.username"/>
			</div>
			<div class="form-group">
				<label>Password</label>
				<input type="password" class="form-control" v-model="loginDTO.password"/>
			</div>
			<button class="btn btn-info btn-lg btn-block" v-on:click="login(loginDTO)">Login</button>
		</div>
		<div v-if="token" class="row">
			<div class="col-md-7">
				<div class="main">
					<div class="text-right">
						<button class="btn btn-outline-info btn-sm" v-on:click="logout()">Logout</button>
					</div>
					<br/>
					<div v-if="error" class="alert alert-danger">
						<span class="fa fa-times"></span> {{ error }}
					</div>
					<div class="row">
						<div class="col-md-4" v-for="item in items" v-bind:key="item.id">
							<div class="card card-body text-center">
								<h5>{{item.description}}</h5>
								<br/>
								<h4>{{formatMoney(item.price)}}</h4>
								<br/>
								<button class="btn btn-info btn-lg" v-on:click="addItem(item)">Add</button>
							</div>
						</div>
					</div>
				</div>
			</div>
			<div class="col-md-5">
				<div class="order">
					<div v-if="!order.code">
						<h5>Place Order</h5>
						<hr/>
						<div class="form-group">
							<label>CPF</label>
							<input type="text" class="form-control" v-model="placeOrderDTO.cpf"/>
						</div>
						<br/>
						<div class="list-group" v-if="placeOrderDTO.items.length === 0">
							<div class="list-group-item text-center">
								There's no item
							</div>
						</div>
						<div class="list-group" v-if="placeOrderDTO.items.length > 0">
							<div class="list-group-item" v-for="item in placeOrderDTO.items" v-bind:key="item.idItem">
								<div class="row">
									<div class="col-md-1">
										{{item.quantity}}
									</div>
									<div class="col-md-6">
										{{item.description}}
									</div>
									<div class="col-md-3 text-right">
										{{formatMoney(item.price * item.quantity)}}
									</div>
									<div class="col-md-2 text-right">
										<button class="btn btn-info btn-sm" v-on:click="removeItem(item)">remove</button>
									</div>
								</div>
							</div>
						</div>
						<br/>
						<div class="form-group">
							<label>Coupon</label>
							<input type="text" class="form-control" v-model="placeOrderDTO.coupon"/>
						</div>
						<br/>
						<button class="btn btn-info btn-lg btn-block" v-on:click="placeOrder(placeOrderDTO)">Confirm ({{formatMoney(getTotal())}})</button>
						<br/>
					</div>
					<div v-if="order.code">
						<h5>Order Success</h5>
						<hr/>
						<div class="list-group">
							<div class="list-group-item">
								Code: {{order.code}}<br/>
								Taxes: {{formatMoney(order.taxes)}}<br/>
								Freight: {{formatMoney(order.freight)}}<br/>
								Amount: {{formatMoney(order.total)}}<br/>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import "bootstrap/dist/css/bootstrap.css";
import "font-awesome/css/font-awesome.css";
import moment from "moment/src/moment.js";
import axios from "axios/dist/axios.js";

export default {
	name: "app",
	data() {
		return {
			items: [],
			error: "",
			token: "",
			placeOrderDTO: {
				items: [],
				issueDate: new Date(),
				cpf: "778.278.412-36",
				zipcode: "11.111-11"
			},
			loginDTO: {
			},
			order: {}
		};
	},
	methods: {
		formatMoney(value) {
			const formatter = new Intl.NumberFormat("pt-BR", {
				style: "currency",
				currency: "BRL"
			});
			return formatter.format(value);
		},
		formatDate(value) {
			const date = new Date(value);
			return `${date.getDate()}/${date.getMonth() + 1}/${date.getFullYear()}`;
		},
		translate(value) {
			const translate = {
				active: "Ativo",
				cancelled: "Cancelado",
				open: "Aberto",
				overdue: "Atrasado",
				paid: "Pago"
			};
			return translate[value];
		},
		addItem(item) {
			const existingItem = this.placeOrderDTO.items.find(existingItem => existingItem.idItem === item.id);
			if (existingItem) {
				existingItem.quantity++;
			} else {
				this.placeOrderDTO.items.push({
					idItem: item.id,
					description: item.description,
					price: item.price,
					quantity: 1
				});
			}
		},
		removeItem(item) {
			const existingItem = this.placeOrderDTO.items.find(existingItem => existingItem.idItem === item.idItem);
			if (existingItem) {
				existingItem.quantity--;
			}
			if (existingItem.quantity === 0) {
				const position = this.placeOrderDTO.items.indexOf(existingItem);
				this.placeOrderDTO.items.splice(position, 1);
			}
		},
		getTotal() {
			let total = 0;
			for (const item of this.placeOrderDTO.items) {
				total += item.price * item.quantity;
			}
			return total;
		},
		placeOrder(placeOrderDTO) {
			this.error = "";
			axios({
				url: "http://localhost:3000/orders",
				method: "post",
				headers: {
					"authentication": "123456"
				},
				data: placeOrderDTO
			}).then(result => {
				this.order = result.data;
					this.placeOrderDTO = {
					items: [],
					issueDate: new Date(),
					cpf: "778.278.412-36",
					zipcode: "11.111-11"
				};
			}).catch(error => {
				this.error = error.response.data.message;
			});
		},
		getItems() {
			axios({
				url: "http://localhost:3000/items",
				method: "get",
				headers: {
					"authentication": "123456"
				}
			}).then(result => {
				const items = result.data;
				this.items = items;
			});
		},
		login(loginDTO) {
			this.error = "";
			axios({
				url: "http://localhost:3000/login",
				method: "post",
				data: loginDTO
			}).then(result => {
				this.token = result.data;
				localStorage.setItem("token", result.data);
			}).catch(error => {
				this.error = error.response.data.message;
			});
		},
		logout() {
			localStorage.removeItem("token");
			location.reload();
		}
	},
	created() {
		this.getItems();
		this.token = localStorage.getItem("token");
	}
};
</script>

<style>

body {
	overflow-x: hidden;
}

.main {
	padding: 30px;
}

.order {
	background-color: #F1F1F1;
	padding: 30px;
	height: 100vh;
}

.well {
	background-color: #F1F1F1;
	padding: 30px;
}

.login {
	width: 400px;
	margin-top: 200px;
	margin-left: auto;
	margin-right: auto;
}

</style>
