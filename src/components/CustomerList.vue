<template>
  <main>
    <br />
    <v-container fluid grid-list-md>
      <v-layout column align-left>
        <blockquote>
          Welcome {{validUserName}}!
          <footer>
            <small>
              <em>&mdash;Eagle Financial Services, your Midwest Financial Services Partner.</em>
            </small>
          </footer>
        </blockquote>
      </v-layout>

      <v-layout column align-center>
        <v-flex xs6 sm8 md7>
          <v-alert
            v-if="showMsg === 'new'"
            dismissible
            :value="true"
            type="success"
          >New customer has been added.</v-alert>
          <v-alert
            v-if="showMsg === 'update'"
            dismissible
            :value="true"
            type="success"
          >Customer information has been updated.</v-alert>
          <v-alert
            v-if="showMsg === 'deleted'"
            dismissible
            :value="true"
            type="success"
          >Selected Customer has been deleted.</v-alert>
        </v-flex>
      </v-layout>
      <br />
      <v-container fluid grid-list-md fill-height>
        <v-layout column>
          <v-flex md6>
            <v-data-table
              :headers="headers"
              :items="customers"
              hide-default-footer
              class="elevation-1"
              fixed
              style="max-height: 300px; overflow-y: auto"
            >
              <template v-slot:item.actions="{ item }">
                <v-icon @click="updateCustomer(item)">mdi-pencil</v-icon>
                <v-icon @click="deleteCustomer(item)">mdi-trash-can</v-icon>
              </template>
            </v-data-table>
          </v-flex>
        </v-layout>
      </v-container>

      <v-btn class="blue white--text" @click="addNewCustomer">Add Customer</v-btn>
    </v-container>
  </main>
</template>


<script>
import router from "../router";
import { APIService } from "../http/APIService";
const apiService = new APIService();

export default {
  name: "CustomerList",
  data: () => ({
    customers: [],
    validUserName: "Guest",
    customerSize: 0,
    showMsg: "",
    headers: [
      { text: "Record Number", value:"pk", sortable: false, align: "left" },
      { text: "Customer Number", value:"cust_number", align: "left", sortable: false },
      { text: "Name", value:"name", sortable: false, align: "left" },
      { text: "Address", value:"address", sortable: false, align: "left" },
      { text: "City", value:"city", sortable: false, align: "left" },
      { text: "State", value:"state", sortable: false, align: "left" },
      { text: "ZipCode", value:"zipcode", sortable: false, align: "left" },
      { text: "Email", value:"email", sortable: false, align: "left" },
      { text: "Phone", value:"cell_phone", sortable: false, align: "left" },
      { text:"Actions", name: "Actions", value: "actions" }
    ]
  }),
  mounted() {
    this.getCustomers();
    this.showMessages();
  },
  methods: {
    showMessages() {
      console.log(this.$route.params.msg);
      if (this.$route.params.msg) {
        this.showMsg = this.$route.params.msg;
      }
    },
    getCustomers() {
      apiService
        .getCustomerList()
        .then(response => {
          this.customers = response.data.data;
          this.customerSize = this.customers.length;
          if (
            localStorage.getItem("isAuthenticates") &&
            JSON.parse(localStorage.getItem("isAuthenticates")) === true
          ) {
            this.validUserName = JSON.parse(localStorage.getItem("log_user"));
          }
        })
        .catch(error => {
          if (error.response.status === 401) {
            localStorage.removeItem("isAuthenticates");
            localStorage.removeItem("log_user");
            localStorage.removeItem("token");
            router.push("/auth");
          }
        });
    },
    addNewCustomer() {
      if (
        localStorage.getItem("isAuthenticates") &&
        JSON.parse(localStorage.getItem("isAuthenticates")) === true
      ) {
        router.push("/customer-create");
      } else {
        router.push("/auth");
      }
    },
    updateCustomer(customer) {
      router.push("/customer-create/" + customer.pk);
    },
    deleteCustomer(customer) {
      apiService
        .deleteCustomer(customer.pk)
        .then(response => {
          if (response.status === 204) {
            alert("Customer deleted");
            this.showMsg = "deleted";
            this.$router.go();
          }
        })
        .catch(error => {
          if (error.response.status === 401) {
            localStorage.removeItem("isAuthenticates");
            localStorage.removeItem("log_user");
            localStorage.removeItem("token");
            router.push("/auth");
          }
        });
    }
  }
};
</script>
