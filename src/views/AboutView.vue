<template>
  <div>
    <Grid
      :data-items="products"
      :filterable="filterable"
      :filter="filter"
      @filterchange="filterChange"
      :columns="columns"
    >
    </Grid>
  </div>
</template>
<script>
import { Grid } from "@progress/kendo-vue-grid";
import { filterBy } from "@progress/kendo-data-query";
import invoices from "../appData/invoices.json";

export default {
  components: {
    Grid: Grid,
  },
  data: function () {
    return {
      filterable: true,
      loader: false,
      filter: null,
    };
  },
  computed: {
    products: function () {
      return filterBy(invoices, this.filter);
    },
    columns: function () {
      return [
        { field: "InvoiceRef", title: "Ref" },
        {
          field: "InvoiceDate",
          filter: "date",
          title: "Date",
          type: "date",
          format: "{0:yyyy-MM-dd}",
        },
        {
          field: "PostingDate",
          filter: "date",
          title: "Submitted",
          type: "date",
          format: "{0:yyyy-MM-dd}",
        },
        { field: "SupplierName", title: "Vendor" },
        { field: "ProcessorName", title: "Financial Assistant" },
        {
          field: "CurrentStateName",
          title: "Current State",
          
        },
        { field: "total", filter: "numeric", title: "Total", format: "{0:c2}" },
      ];
    },
  },
  methods: {
    filterChange: function (ev) {
      this.filter = ev.filter;
    },
  },
};
</script>
