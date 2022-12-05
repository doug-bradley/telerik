<template>

    <card :orientation="'horizontal'" class="card">
        <grid 
                :data-items="result" 
                :columns="gridColumns"

                :filterable="filterable"
                :filter="filter"
                @filterchange="filterChange"

                :pageable="pageable"
                :page-size="paging.size"
                :skip="paging.skip"
                :take="paging.take"
                @pagechange="onPageChange"
                :total="total"
                
                :loader="loader"
            >
            <template v-slot:rowActions="{props}">
                <custom :data-item="props.dataItem" 
                        @edit="edit"
                        @remove="remove"/>
            </template>
            <template v-slot:stateColor="{ props }">
                <td :class="props.className" v-html="getNestedValue(props.field, props.dataItem)"></td>
            </template>

            <toolbar>
                <kendo-button>New</kendo-button>
                <kendo-button @click="(filterable = !filterable)">Filter</kendo-button>
                <kendo-button  @click="exportExcel">Export</kendo-button>
                <drop-button :item-render="itemRender" :text="'Columns'" :items="gridColumns" :text-field="'title'" :fill-mode="'outline'">
                    <template v-slot:itemRender="{ props }">
                        <div class="k-link k-menu-link">
                            <!-- <input :id="'column-visibility-show-' + idx"
                                        :class="'k-checkbox k-checkbox-md k-rounded-md'" :type="'checkbox'"
                                        :readOnly="true" :disabled="!column.hidden && oneVisibleColumn"
                                        :checked="!column.hidden" @click="onToggleColumn(idx)" /> -->
                        <span :class="`k-icon k-i-${props.item.title}`" role="presentation" />
                        <em>{{ `action #${props.itemIndex}: ${props.item.title}` }}</em>
                        </div>
                    </template>
                </drop-button>
            </toolbar>

        </grid>
        <grid-editor v-if="productInEdit" :data-item="productInEdit" @save="save" @cancel="cancel">
        </grid-editor>
    </card>
</template>

<script>
import axios from 'axios';
import { reduce } from 'lodash';

import { Grid, GridToolbar } from '@progress/kendo-vue-grid';
import { Card } from '@progress/kendo-vue-layout';
import { Button } from '@progress/kendo-vue-buttons';
import { DropDownButton } from "@progress/kendo-vue-buttons";

import { filterBy } from "@progress/kendo-data-query";
import CommandCell from '../components/FinalGridActionCell.vue';

import { saveExcel } from '@progress/kendo-vue-excel-export';
import GridEditor from '../components/GridEditor.vue';

export default {
    components: {
        grid: Grid, 
        toolbar: GridToolbar,
        card: Card,
        kendoButton: Button,
        'grid-editor': GridEditor,
        custom: CommandCell,
        'drop-button': DropDownButton
    },
    created: function () {
        this.getInvoices();
     },
    data: function () {
        return {
            itemRender: 'itemRender',
            filterable: false,
            filter: null,
            loader: false,
            
            paging: {
                skip: 0,
                take: 10,
                size: 10,
            },

            gridItems: [],
            gridColumns: [
                {
                    field: "InvoiceRef",
                    title: "Ref",
                    locked: false,
                    //columnMenu: "myTemplate",
                    hidden: false,
                },
                {
                    field: "InvoiceDate",
                    filter: "date",
                    title: "Date",
                    type: "date",
                    format: "{0:yyyy-MM-dd}",
                    locked: false,
                    //columnMenu: "myTemplate",
                    hidden: false,
                },
                {
                    field: "PostingDate",
                    filter: "date",
                    title: "Submitted",
                    type: "date",
                    format: "{0:yyyy-MM-dd}", locked: false,
                    //columnMenu: "myTemplate",
                    hidden: false,
                    width: 200,
                },
                {
                    field: "SupplierName", title: "Vendor", locked: false,
                    //columnMenu: "myTemplate",
                    hidden: false,
                    width: 200,
                },
                {
                    field: "ProcessorName", title: "Financial Assistant", locked: false,
                    //columnMenu: "myTemplate",
                    hidden: false,
                    width: 200,
                },
                {
                    field: "CurrentStateName",
                    title: "Current State", locked: false,
                    //columnMenu: "myTemplate",
                    hidden: false,
                    width: 200,
                    cell: 'stateColor'
                },
                {
                    field: "total", filter: "numeric", title: "Total", format: "{0:c2}", locked: false,
                    //columnMenu: "myTemplate",
                    hidden: false,
                    width: 200,
                },
                { 
                    cell: 'rowActions',
                    width: '210px',
                    filterable: false, 
                    columnMenu: false,
                }
            ],
            productInEdit: undefined,
        };
    },  
    computed: {
        pageable() {
            return {
                buttonCount: 5,
                info: true,
                type: 'numeric',
                pageSizes: [5, 10, 15, 20, 'all'],
                previousNext: true,
                pageSizeValue: this.paging.size,
            };
        },
        result: function () {
                
            const filtered =  filterBy(this.gridItems, this.filter);
            return filtered.slice(this.paging.skip, this.paging.take + this.paging.skip);
        },
        total() {
            return this.gridItems ? this.gridItems.length : 0;
        },
    },
    methods: {
        exportExcel() {
            const columnsToExport = [...this.gridColumns];
            columnsToExport.splice(0, 1);
            saveExcel({
                data: this.gridItems,
                group: this.group,
                fileName: 'invoices',
                columns: columnsToExport,
            });
        },
        filterChange: function (ev) {
            this.filter = ev.filter;
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
        edit(dataItem) {
          this.productInEdit = this.cloneProduct(dataItem);
       },
       remove(dataItem) {
          this.products = this.products.filter(p => p.ProductID !== dataItem.ProductID);
      },
       save() {
        const dataItem = this.productInEdit;
        const products = this.products.slice();
        const isNewProduct = dataItem.ProductID === undefined;

        if (isNewProduct) {
            products.unshift(this.newProduct(dataItem));
        } else {
            const index = products.findIndex(p => p.ProductID === dataItem.ProductID);
            products.splice(index, 1, dataItem);
        }

        this.products= products;
        this.productInEdit= undefined;
      },
      cancel () {
          this.productInEdit= undefined;
      },
      insert () {
          this.productInEdit = { };
      },
      dialogTitle() {
          return `${this.productInEdit.ProductID === undefined ? 'Add' : 'Edit'} product`;
      },
      cloneProduct(product) {
          return Object.assign({}, product);
      },
      newProduct(source) {
          const id = this.products.reduce((acc, current) => Math.max(acc, current.ProductID || 0), 0) + 1;
          const newProduct = {
              ProductID: id,
              ProductName: '',
              UnitsInStock: 0,
              Discontinued: false
          };

          return Object.assign(newProduct, source);
      },
        onPageChange(event) {
            this.loader = true;
            
            setTimeout(() => {
                this.loader = false;
                this.paging.skip = event.page.skip;
                this.paging.take = event.event.value === 'all' ? 1000 : event.page.take;

                this.paging.size = event.event.value;
            }, 300);
        },
        getInvoices() {
            axios.get('https://fxvue-api.herokuapp.com/invoices')
                .then((response) => {
                    response.data.forEach(invoice => {
                        invoice.total = reduce(invoice.items, (sum, item) => sum + item.InclItemCost, 0);
                    });
                    this.gridItems = response.data;
                })
                .catch((error) => {
                    console.log(error);
                });
            },
        },

};
</script>


<style>
.card {
    margin: 35px;
    padding: 25px
}
.k-toolbar.k-grid-toolbar {
    justify-content: flex-end !important;
}
</style>