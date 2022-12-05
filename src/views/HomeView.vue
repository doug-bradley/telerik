<template>
  <div id="app">
    <card :orientation="'horizontal'" style="margin: 35px; padding: 25px">

      <grid 
        :pageable=true 
        :skip="skip"
        :take="take"
        :loader="loader"
        :sortable="sortable" cc
        @filterchange="filterChange"
        :data-items="gridInvoices" 
        :columns="invoiceColumns" 
        @custom="customHandler"
        :total="total"
    @pagechange="onPageChange"
        >

        <toolbar>
          <kendo-button title="New" :theme-color="'primary'" @click="exportExcel">New</kendo-button>
          <kendo-button title="Filter" :theme-color="'primary'"
            @click="(filterable = !filterable)">Filter</kendo-button>
          <kendo-button title="Customize" :theme-color="'primary'" @click="exportExcel">Customize</kendo-button>
          <kendo-button title="Export to Excel" :theme-color="'primary'" @click="exportExcel">{{ exportExcelMessage }}
          </kendo-button>
        </toolbar>

        <template v-slot:myTemplate="{ props }">
          <td :class="props.className" v-html="getNestedValue(props.field, props.dataItem)"></td>
        </template>

      </grid>
    </card>
  </div>
</template>

<script>
import '@progress/kendo-theme-default/dist/all.css';
import { Grid, GridToolbar } from '@progress/kendo-vue-grid';
import axios from 'axios';
import { Card } from '@progress/kendo-vue-layout';
import { saveExcel } from '@progress/kendo-vue-excel-export';
import { Button } from '@progress/kendo-vue-buttons';
import { filterBy } from "@progress/kendo-data-query";
import invoices from '../appData/invoices.json';

export default {
  name: 'App',
  components: {
    grid: Grid,
    toolbar: GridToolbar,
    card: Card,
    kendoButton: Button,
  },
  data: function () {
    return {
      invoices,
      filterable: false,
      pageable: true,
      loader: false,
      skip: 0,
      take: 10,
      sortable: true,
      filter: null,
    };
  },
  computed: {
    invoiceColumns: function () {
      return [
        { field: 'InvoiceRef', title: 'Ref' },
        { field: 'InvoiceDate', filter: "date", title: 'Date', type: "date", format: "{0:yyyy-MM-dd}" },
        { field: 'PostingDate', filter: "date", title: 'Submitted', type: "date", format: "{0:yyyy-MM-dd}" },
        { field: 'SupplierName', title: 'Vendor' },
        { field: 'ProcessorName', title: 'Financial Assistant' },
        { field: 'CurrentStateName', title: 'Current State', cell: 'myTemplate' },//, cell: this.cellFunction
        { field: 'total', filter: "numeric", title: 'Total', format: "{0:c2}" },
      ];
    },
    gridInvoices: function () {
      console.log('invoices', this.invoices);
      return filterBy(this.invoices.slice(this.skip, this.take + this.skip), this.filter);
    },
    exportExcelMessage() {
      return 'Export to Excel'
    },
    total() {
      return this.invoices ? this.invoices.length : 0;
    },
  },
  methods: {
    onPageChange(event) {
      this.loader = true;
      // The idea behind using the following setTimeout method is to
      // demonstrate how we can show a loader while fetching data.
      // In a real-life scenarios with remote data binding, the 'loader'
      // property should be updated before making a server request
      // and after the request completes.
      setTimeout(() => {
        this.loader = false;
        this.skip = event.page.skip;
        this.take = event.page.take;
      }, 300);
    },
    exportExcel() {
      const columnsToExport = [...this.invoiceColumns];
      columnsToExport.splice(0, 1);
      saveExcel({
        data: this.gridInvoices,
        group: this.group,
        fileName: 'Employees',
        columns: columnsToExport,
      });
    },

    filterChange: function (ev) {
      this.filter = ev.filter;
    },
    customHandler: function (e) {
      console.log('customHandler', e);
    },
    cellFunction: function (h, tdElement, props, listeners) {
      return h('td', {
        on: {
          click: function (e) {
            listeners.custom(e);
          }
        }
      }, [props.dataItem.CurrentStateName]);
    },
    getNestedValue: function (fieldName, dataItem) {
      const path = fieldName.split('.');
      let data = dataItem;
      path.forEach((p) => {
        data = data ? data[p] : undefined;
      });
      var color;
      switch (data) {
        case 'Closed':
          color = 'red';
          break;
        case 'Create':
          color = 'green';
          break;
        case 'ApproveCC':
          color = 'orange';
          break;
        case 'Cancelled':
          color = 'blue';
          break;
        default:
          color = 'purple';
          break;
      }
      return `<div style="color: ${color}"> ${data}</div>`;
    },
    getInvoices() {
      axios
        .get('https://fxvue-api.herokuapp.com/invoices')
        .then((response) => {
          // response.data.forEach(invoice => {
          //   invoice.total = reduce(invoice.items, (sum, item) => sum + item.InclItemCost, 0);
          // });
          this.invoices = response.data;
        })
        .catch((error) => {
          console.log(error);
        });
    },

  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;

  margin-top: 60px;
}
</style>
