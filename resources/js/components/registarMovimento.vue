<template>
    <div class="container">
        <form>
            <div class="jumbotron" style="background-color: rgb(48,117,173);">
                <h1 style="text-align: center"><span style="color: white; ">{{title}}</span></h1>
            </div>
            <div class="col-md-12">
                <label>Email</label>
                <vue-select @class="form-control" name="email" :filterable="false" :options="emails"
                            @search="fetchEmails"
                            v-model="$v.email.$model" placeholder="Insert an email"></vue-select>
                <div style="color:red" v-if="!$v.email.required && $v.email.$anyDirty">Email is required</div>
                <div style="color:red" v-if="!$v.email.email && $v.email.$anyDirty">Invalid format of email</div>
            </div>
            <div class="col-md-12">
                <br>
                <label>Value of income</label>
                <input type="number" class="form-control" id="value" name="value" v-model="$v.value.$model"
                       placeholder="Value of income">
                <div style="color:red" v-if="!$v.value.required && $v.value.$anyDirty">Value is required</div>
                <div style="color:red" v-if="!$v.value.minValue && $v.value.$anyDirty">Value must be greater than 0.01</div>
                <div style="color:red" v-if="!$v.value.maxValue && $v.value.$anyDirty">Value must be less than 5000.00</div>
            </div>
            <div class="col-md-12">
                <br>
                <label>Type of payment</label>
                <select name="type_payment" id="tPagamento" class="form-control" v-model="$v.type_payment.$model">
                    <option disabled selected> -- Select an option --</option>
                    <option value="c">Cash</option>
                    <option value="bt">Bank Transfer</option>
                </select>
                <div style="color:red" v-if="!$v.type_payment.required && $v.type_payment.$anyDirty">Type of payment is required</div>
            </div>
            <div class="col-md-12">
                <br>
                <label>Source Description</label>
                <input type="text" class="form-control" id="source_description" name="source_description"
                       placeholder="Source Description" v-model="source_description">
            </div>
            <div class="col-md-12" v-if="type_payment=='bt'">
                <br>
                <label>IBAN</label>
                <input type="text" class="form-control" id="iban" name="iban" placeholder="IBAN" v-model="$v.iban.$model">
                <div style="color:red" v-if="!$v.iban.maxLength && $v.iban.$anyDirty">IBAN must have 25 digits</div>
            </div>
            <div class="col-md-12">
                <br>
                <button @click.prevent="!$v.$invalid ? registerMovement() : showErrors()" class="btn btn-primary">Add Income</button>
            </div>
        </form>

    </div>
</template>

<script>
    import vueSelect from 'vue-select';
    import 'vue-select/dist/vue-select.css';
    import {required,numeric,email,minLength,maxLength,minValue,maxValue,helpers} from "vuelidate/src/validators";

    export default {
        name: "registarMovimento",
        components: {
            vueSelect,
        },
        data: function () {
            return {
                title: 'Regist Income',
                emails: [],
                email: null,
                value: "",
                type_payment: "",
                source_description: "",
                iban: "",
            };
        },
        methods: {
            registerMovement() {
                axios.get('/api/wallets/id/' + this.email)
                    .then(response => {
                        axios.post('/api/movements', {
                            email: this.email,
                            value: this.value,
                            type_payment: this.type_payment,
                            source_description: this.source_description,
                            iban: this.iban,
                            wallet_id: response.data[0].id,
                            start_balance: response.data[0].balance,
                            //end_balance: Number(response.data[0].balance)+Number(this.value), //Não é seguro estar aqui
                        })
                        .then(async (response) => {
                            await axios.get('/api/wallets/id/' + this.email)
                                .then(response => {
                                    this.$socket.emit('balance-user',response.data[0].balance, response.data[0].id);
                                    if(this.value > 3000){
                                        this.$socket.emit('notificacao-operador',response.data[0].balance, response.data[0].id);
                                    }
                                })
                                .catch(error => {
                                    console.log(error);
                                });
                            this.$router.push('/');
                        })
                        .catch(error => {
                            console.log(error);
                        });
                    }).catch(error => {
                        console.log(error);
                    });
            },
            showErrors(){
                this.$v.$touch();
            },
            fetchEmails(search, loading) {
                let self = this;
                loading(true);
                if(search){
                    axios.get('/api/wallets/email/' + search)
                        .then(function (response) {
                            self.emails = response.data;
                        })
                        .catch(function (error) {
                            // console.log(error);
                        })
                        .finally(function () {
                            loading(false);
                        });
                }
            },
        },validations: {
            email: {
                required, email
            },
            iban: {
                minLength:minLength(25), maxLength:maxLength(25)
            },
            value: {
                required, minValue:minValue(0.01), maxValue:maxValue(5000.00)
            },
            type_payment: {
                required
            },
        }
    }
</script>

<style scoped>

</style>
         
