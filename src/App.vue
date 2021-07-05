<template>
	<div>
		<div class="row">
			<div class="menu col-md-7">
				<div class="list-group">
					<div class="list-group-item" v-for="enrollment in enrollments" v-bind:key="enrollment.code">
						<div class="row">
							<div class="col-md-10">
								{{ enrollment.code }}
							</div>
							<div class="col-md-2 text-right">
								<button class="btn btn-outline-info btn-sm" v-on:click="cancelEnrollment(enrollment)">
									<span class="fa fa-times"></span>
								</button>
							</div>
						</div>
					</div>
				</div>
			</div>
			<div class="order col-md-5">
				<h5>Matrícula</h5>
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
		enrollStudent(enrollStudentDTO) {
			axios({
				url: "http://localhost:3000/enrollments",
				method: "post",
				headers: {
					"authentication": "123456"
				},
				data: enrollStudentDTO
			}).then(result => {
				this.getEnrollments();
			});
		},
		cancelEnrollment(getEnrollmentDTO) {
			console.log(getEnrollmentDTO);
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
.menu {
	padding: 30px;
}
.order {
	background-color: #F1F1F1;
	padding: 30px;
	height: 100vh;
}
.no-items {
	padding-top: 50px;
	padding-bottom: 50px;
}
</style>
