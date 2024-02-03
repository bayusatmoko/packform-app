<template>
  <v-card
    flat
  >
    <template v-slot:text>
      <v-text-field
        v-model="keyword"
        label="Search"
        prepend-inner-icon="mdi-magnify"
        single-line
        variant="outlined"
        hide-details
        @input="fetchOrders({page: this.page})"
      ></v-text-field>
      <v-text-field 
        label="Date"
        variant="outlined"
        @click:control="onDateInput(true)"
        v-model="dateRange"
      ></v-text-field>
    </template>
    <v-date-picker
      v-model="date"
      multiple=2
      v-show="showDate"
    >
    </v-date-picker>

    <v-data-table-server
    :items-per-page=5
    :headers="headers"
    :items-length="rows"
    :items="orders"
    :loading="loading"
    item-value="name"
    v-model:page="page"
    @update:options="fetchOrders({page: this.page})"
  ></v-data-table-server>
  </v-card>
</template>

<script>
import axios from "axios";

  export default {
    data () {
      return {
        keyword: '',
        date: [],
        rows: 5,
        page: 1,
        showDate: false,
        loading: false,
        headers: [
          {
            align: 'start',
            key: 'Order_name',
            sortable: false,
            title: 'Order name',
          },
          { key: 'Company_name', title: 'Customer Company', sortable: false },
          { key: 'Name', title: 'Customer name', sortable: false },
          { key: 'Created_at', title: 'Order date' },
          { key: 'Delivered_amount', title: 'Delivered Amount', sortable: false },
          { key: 'Total_amount', title: 'Total Amount', sortable: false },
        ],
        orders: [],
      }
    },
    watch: {
      date: function () {
        if(this.date.length == 2) {
          this.showDate = false
          this.fetchOrders({page: this.page})
        }
      },
    },
    methods: {
      onDateInput(val) {
        this.showDate = val
        this.date = []
      },
      async fetchOrders() {
        this.loading = true
        let params = {
          url: "http://localhost:8080/orders",
          method: "get",
          params: {
            keyword: this.keyword,
            startDate: '1970-01-01',
            endDate: '3000-01-01',
            offset: (this.page - 1) * 5,
          },
        };
        if (this.date.length > 0) {
          params.params.startDate = JSON.stringify(new Date(this.date[0])).slice(1,11)
          params.params.endDate = JSON.stringify(new Date(this.date[1])).slice(1,11)
        }
        const response = await axios(params);
        this.orders = response.data.orders;
        if(!this.orders) this.orders = []
        this.orders.forEach((order) => {
          order.Order_name = order.Order_name + ' : ' + order.Product
          if(!order.Delivered_amount) order.Delivered_amount = '-'
        })
        this.rows = response.data.rows;
        this.loading = false
      },
    },
    computed: {
      dateRange: function () {
        if(this.date.length > 0) {
          return this.date[0] + ' - ' + this.date[1]
        } else {
          return 'Select date range . . .'
        }
      }
    },
    async mounted() {
      await this.fetchOrders({page: this.page});
    },
  }
</script>