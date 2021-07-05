<template>
	<div>
		<div class="row">
			<div class="col-md-8">
				<div class="main">
					<div v-if="error" class="alert alert-danger">
						<span class="fa fa-times"></span> {{ error }}
					</div>
					<div v-if="enrollments.length === 0">
						<div class="text-center" style="padding-top: 300px;">
							<span class="fa fa-list fa-3x text-muted"></span>
							<br/><br/>
							<h5>Nenhuma Matrícula</h5>
						</div>
					</div>
					<div class="list-group" v-for="enrollment in enrollments" v-bind:key="enrollment.code">
						<div class="list-group-item" >
							<div class="row">
								<div class="col-md-12">
									<div class="well">
										<div class="row">
											<div class="col-md-4 text-center">
												<h5>Código</h5>
												{{ enrollment.code }}<br/>
											</div>
											<div class="col-md-4 text-center">
												<h5>Aluno</h5>
												{{ enrollment.studentName }}<br/>
											</div>
											<div class="col-md-4 text-center">
												<h5>CPF</h5>
												{{ enrollment.studentCpf}}<br/>
											</div>
										</div>
										<br/>
										<div class="row">
											<div class="col-md-4 text-center">
												<h5>Dt. Nascimento</h5>
												{{ formatDate(enrollment.studentBirthDate) }}<br/>
											</div>
											<div class="col-md-4 text-center">
												<h5>Nível</h5>
												{{ enrollment.levelDescription }}<br/>
											</div>
											<div class="col-md-4 text-center">
												<h5>Módulo</h5>
												{{ enrollment.moduleDescription}}<br/>
											</div>
										</div>
										<br/>
										<div class="row">
											<div class="col-md-4 text-center">
												<h5>Turma</h5>
												{{ enrollment.classroomCode}}<br/>
											</div>
											<div class="col-md-4 text-center">
												<h5>Status</h5>
												{{ translate(enrollment.status) }}<br/>
											</div>
											<div class="col-md-4 text-center">
												<h5>Saldo</h5>
												{{ formatMoney(enrollment.balance)}}<br/>
											</div>
										</div>
										<br/>
										<div class="text-center">
											<button class="btn btn-outline-danger btn-sm" v-on:click="cancelEnrollment(enrollment)">
												<span class="fa fa-times"></span> Cancelar Matrícula
											</button>
										</div>
									</div>
									<br/>
									<table class="table table-striped">
										<thead>
											<tr>
												<th>Ciclo</th>
												<th>Vencimento</th>
												<th>Valor</th>
												<th>Multa</th>
												<th>Juros</th>
												<th>Saldo</th>
												<th>Status</th>
												<th></th>
											</tr>
										</thead>
										<tbody>
											<tr v-for="invoice in enrollment.invoices" v-bind:key="invoice.month">
												<td>{{invoice.month}}/{{invoice.year}}</td>
												<td>{{formatDate(invoice.dueDate)}}</td>
												<td>{{formatMoney(invoice.amount)}}</td>
												<td>{{formatMoney(invoice.penalty)}}</td>
												<td>{{formatMoney(invoice.interests)}}</td>
												<td>{{formatMoney(invoice.balance)}}</td>
												<td>{{ translate(invoice.status) }}</td>
												<td>
													<button class="btn btn-outline-info btn-sm" v-on:click="payInvoice(enrollment, invoice)">
														<span class="fa fa-dollar"></span> Pagar
													</button>
												</td>
											</tr>
										</tbody>
									</table>									
								</div>
							</div>
						</div>
						<br/>
					</div>
				</div>
			</div>
			<div class="col-md-4">
				<div class="enrollment">
					<h5>Matricular Aluno</h5>
					<hr/>
					<div class="form-group">
						<label>Nome</label>
						<input type="text" class="form-control" v-model="enrollStudentDTO.studentName"/>
					</div>
					<div class="form-group">
						<label>CPF</label>
						<input type="text" class="form-control" v-model="enrollStudentDTO.studentCpf"/>
					</div>
					<div class="form-group">
						<label>Data de Nascimento</label>
						<input type="text" class="form-control" v-model="enrollStudentDTO.studentBirthDate"/>
					</div>
					<div class="form-group">
						<label>Nível</label>
						<input type="text" class="form-control" v-model="enrollStudentDTO.level"/>
					</div>
					<div class="form-group">
						<label>Módulo</label>
						<input type="text" class="form-control" v-model="enrollStudentDTO.module"/>
					</div>
					<div class="form-group">
						<label>Turma</label>
						<input type="text" class="form-control" v-model="enrollStudentDTO.classroom"/>
					</div>
					<div class="form-group">
						<label>Quantidade de Parcelas</label>
						<input type="text" class="form-control" v-model="enrollStudentDTO.installments"/>
					</div>
					<br/>
					<button class="btn btn-info btn-lg btn-block" v-on:click="enrollStudent(enrollStudentDTO)">Confirmar</button>
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
			enrollments: [],
			error: "",
			enrollStudentDTO: {
			}
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
		enrollStudent(enrollStudentDTO) {
			this.error = "";
			axios({
				url: "http://localhost:3000/enrollments",
				method: "post",
				headers: {
					"authentication": "123456"
				},
				data: enrollStudentDTO
			}).then(result => {
				this.enrollStudentDTO = {};
				this.getEnrollments();
			}).catch(error => {
				this.error = error.response.data.message;
			});
		},
		cancelEnrollment(getEnrollmentDTO) {
			this.error = "";
			axios({
				url: `http://localhost:3000/enrollments/${getEnrollmentDTO.code}/cancel`,
				method: "post",
				headers: {
					"authentication": "123456"
				}
			}).then(result => {
				this.getEnrollments();
			}).catch(error => {
				this.error = error.response.data.message;
			});
		},
		payInvoice(enrollment, invoice) {
			this.error = "";
			axios({
				url: `http://localhost:3000/enrollments/${enrollment.code}/pay`,
				method: "post",
				headers: {
					"authentication": "123456"
				},
				data: {
					code: enrollment.code,
					month: invoice.month,
					year: invoice.year,
					amount: invoice.amount + invoice.penalty + invoice.interests,
					paymentDate: new Date()
				}
			}).then(result => {
				this.getEnrollments();
			}).catch(error => {
				this.error = error.response.data.message;
			});
		},
		getEnrollments() {
			axios({
				url: "http://localhost:3000/enrollments",
				method: "get",
				headers: {
					"authentication": "123456"
				}
			}).then(result => {
				const enrollments = result.data;
				this.enrollments = enrollments;
			});
		}
	},
	created() {
		this.getEnrollments();
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

.enrollment {
	background-color: #F1F1F1;
	padding: 30px;
	height: 100vh;
}

.well {
	background-color: #F1F1F1;
	padding: 30px;
}

</style>
