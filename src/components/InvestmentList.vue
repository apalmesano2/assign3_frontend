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
          >New investment has been added.</v-alert>
          <v-alert
            v-if="showMsg === 'update'"
            dismissible
            :value="true"
            type="success"
          >Investment information has been updated.</v-alert>
          <v-alert
            v-if="showMsg === 'deleted'"
            dismissible
            :value="true"
            type="success"
          >Selected Investment has been deleted.</v-alert>
        </v-flex>
      </v-layout>
      <br />
      <v-container fluid grid-list-md fill-height>
        <v-layout column>
          <v-flex md6>
            <v-data-table
              :headers="headers"
              :items="investments"
              hide-default-footer
              class="elevation-1"
              fixed
              style="max-height: 300px; overflow-y: auto"
            >
              <template v-slot:item.actions="{ item }">
                <v-icon @click="updateInvestment(item)">mdi-pencil</v-icon>
                <v-icon @click="deleteInvestment(item)">mdi-trash-can</v-icon>
              </template>
            </v-data-table>
          </v-flex>
        </v-layout>
      </v-container>

      <v-btn class="blue white--text" @click="addNewInvestment">Add Investment</v-btn>
    </v-container>
  </main>
</template>


<script>
import router from "../router";
import { APIService } from "../http/APIService";
import vuetify from '../plugins/vuetify'
const apiService = new APIService();

export default {
  name: "InvestmentList",
  data: () => ({
    investments: [],
    validUserName: "Guest",
    investmentSize: 0,
    showMsg: "",
    headers: [
      { text: "Customer", value:"customer", sortable: false, align: "left" },
      { text: "Category", value:"category", sortable: false, align: "left" },
      { text: "Description", value:"description", sortable: false, align: "left" },
      { text: "Acquired Value", value:"acquired_value", sortable: false, align: "left" },
      { text: "Acquired Date", value:"acquired_date", sortable: false, align: "left" },
      { text: "Recent Value", value:"recent_value", sortable: false, align: "left" },
      { text: "Recent Date", value:"recent_date", sortable: false, align: "left" },
      { text:"Actions", name:"Actions", value:"actions" }
    ]
  }),
  mounted() {
    this.getInvestments();
    this.showMessages();
  },
  methods: {
    showMessages() {
      console.log(this.$route.params.msg);
      if (this.$route.params.msg) {
        this.showMsg = this.$route.params.msg;
      }
    },
    getInvestments() {
      apiService
        .getInvestmentList()
        .then(response => {
          this.investments = response.data.data;
          this.investmentSize = this.investments.length;
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
    addNewInvestment() {
      if (
        localStorage.getItem("isAuthenticates") &&
        JSON.parse(localStorage.getItem("isAuthenticates")) === true
      ) {
        router.push("/investment-create");
      } else {
        router.push("/auth");
      }
    },
    updateInvestment(investment) {
      router.push("/investment-create/" + investment.pk);
    },
    deleteInvestment(investment) {
      apiService
        .deleteInvestment(investment.pk)
        .then(response => {
          if (response.status === 204) {
            alert("Investment deleted");
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
