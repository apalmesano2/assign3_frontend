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
          >New stock has been added.</v-alert>
          <v-alert
            v-if="showMsg === 'update'"
            dismissible
            :value="true"
            type="success"
          >Stock information has been updated.</v-alert>
          <v-alert
            v-if="showMsg === 'deleted'"
            dismissible
            :value="true"
            type="success"
          >Selected Stock has been deleted.</v-alert>
        </v-flex>
      </v-layout>
      <br />
      <v-container fluid grid-list-md fill-height>
        <v-layout column>
          <v-flex md6>
            <v-data-table
              :headers="headers"
              :items="stocks"
              hide-default-footer
              class="elevation-1"
              fixed
              style="max-height: 300px; overflow-y: auto"
            >
              <template v-slot:item.actions="{ item }">
                <v-icon @click="updateStock(item)">mdi-pencil</v-icon>
                <v-icon @click="deleteStock(item)">mdi-trash-can</v-icon>
              </template>
            </v-data-table>
          </v-flex>
        </v-layout>
      </v-container>

      <v-btn class="blue white--text" @click="addNewStock">Add Stock</v-btn>
    </v-container>
  </main>
</template>


<script>
import router from "../router";
import { APIService } from "../http/APIService";
const apiService = new APIService();

export default {
  name: "StockList",
  data: () => ({
    stocks: [],
    validUserName: "Guest",
    stockSize: 0,
    showMsg: "",
    headers: [
      { text: "Customer", value:"customer", sortable: false, align: "left" },
      { text: "Symbol", value:"symbol", sortable: false, align: "left" },
      { text: "Name", value:"name", sortable: false, align: "left" },
      { text: "Shares", value:"shares", sortable: false, align: "left" },
      { text: "Purchase Price", value:"purchase_price", sortable: false, align: "left" },
      { text: "Purchase Date", value:"purchase_date", sortable: false, align: "left" },
      { text:"Actions", name: "Actions", value: "actions"}
    ]
  }),
  mounted() {
    this.getStocks();
    this.showMessages();
  },
  methods: {
    showMessages() {
      console.log(this.$route.params.msg);
      if (this.$route.params.msg) {
        this.showMsg = this.$route.params.msg;
      }
    },
    getStocks() {
      window.x = this;
      apiService
        .getStockList()
        .then(response => {
          this.stocks = response.data.data;
          this.stockSize = this.stocks.length;
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
    addNewStock() {
      if (
        localStorage.getItem("isAuthenticates") &&
        JSON.parse(localStorage.getItem("isAuthenticates")) === true
      ) {
        router.push("/stock-create");
      } else {
        router.push("/auth");
      }
    },
    updateStock(stock) {
      router.push("/stock-create/" + stock.pk);
    },
    deleteStock(stock) {
      apiService
        .deleteStock(stock.pk)
        .then(response => {
          if (response.status === 204) {
            alert("Stock deleted");
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
