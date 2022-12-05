<template>

    <card :orientation="'horizontal'" style="margin: 35px; padding: 25px">
        <Grid :data-items="products" 
            :take="dataState.take"
            :skip="dataState.skip"
            :sort="dataState.sort"
            :filter="dataState.filter"
            :filterable="filterable"  
            @filterchange="filterChange"
            :columns="columns" 
            @datastatechange="dataStateChange"        
            @columnreorder="handleColumnReorder" 
            :sortable="true"
            :pageable="true" 
            :reorderable="true" 
            :page-size="8">
            <toolbar>
                <kendo-button title="New" :theme-color="'primary'">New</kendo-button>
                <kendo-button title="Filter" :theme-color="'primary'"
                    @click="(filterable = !filterable)">Filter</kendo-button>
                <kendo-button title="Customize" :theme-color="'primary'">Customize</kendo-button>
                <kendo-button title="Export to Excel" :theme-color="'primary'">Export</kendo-button>
            </toolbar>

            <template v-slot:myTemplate="{ props }">
                <div>
                    <custom :column="props.column" 
                    :filterable="props.filterable" 
                    :filter="props.filter"
                        :sortable="props.sortable" 
                        :sort="props.sort" 
                        :columns="columns"
                        :locked="isColumnLocked(props.column.field)"                        
                        @sortchange="(e) => props.onSortchange(e)"
                        @lockchange="(e) => lockChange(e, props.column)"
                        @visibilitychange="(e) => visibilityChange(e, props.column)"
                        @filterchange="(e) => props.onFilterchange(e)" 
                        @closemenu="(e) => props.onClosemenu(e)"
                        @contentfocus="(e) => props.onContentfocus(e)" 
                        @columnssubmit="onColumnsSubmit" />
                </div>
            </template>


        </Grid>
    </card>
</template>
<script>
import { Grid, GridToolbar } from '@progress/kendo-vue-grid';
import { filterBy } from "@progress/kendo-data-query";
import { Card } from '@progress/kendo-vue-layout';
import { process } from "@progress/kendo-data-query";
import { Button } from '@progress/kendo-vue-buttons';
import ColumnMenu from "../components/ColumnMenu.vue";
import products from '../appData/invoices.json';


export default {
    components: {
        Grid: Grid, toolbar: GridToolbar,
        card: Card,
        kendoButton: Button,
        custom: ColumnMenu,
    },
    created: function () {
        this.createDataState({
            take: 8,
            skip: 0,
        });
    },
    data: function () {
        return {
            filterable: false,
            loader: false,
            filter: null,
            result: [],
            dataState: {
                take: 0,
                skip: 0,
            },
            columns: [
                {
                    field: "InvoiceRef",
                    title: "Ref",
                    locked: false,
                    columnMenu: "myTemplate",
                    hidden: false,
                },
                {
                    field: "InvoiceDate",
                    filter: "date",
                    title: "Date",
                    type: "date",
                    format: "{0:yyyy-MM-dd}",
                    locked: false,
                    columnMenu: "myTemplate",
                    hidden: false,
                },
                {
                    field: "PostingDate",
                    filter: "date",
                    title: "Submitted",
                    type: "date",
                    format: "{0:yyyy-MM-dd}", locked: false,
                    columnMenu: "myTemplate",
                    hidden: false,
                    width: 100,
                },
                {
                    field: "SupplierName", title: "Vendor", locked: false,
                    columnMenu: "myTemplate",
                    hidden: false,
                    width: 100,
                },
                {
                    field: "ProcessorName", title: "Financial Assistant", locked: false,
                    columnMenu: "myTemplate",
                    hidden: false,
                    width: 100,
                },
                {
                    field: "CurrentStateName",
                    title: "Current State", locked: false,
                    columnMenu: "myTemplate",
                    hidden: false,
                    width: 100,

                },
                {
                    field: "total", filter: "numeric", title: "Total", format: "{0:c2}", locked: false,
                    columnMenu: "myTemplate",
                    hidden: false,
                    width: 100,
                },
            ]
        };
    },
    computed: {
        products: function () {
            console.log(products, this.filter)
            return filterBy(products, this.filter);
        },
        lockedColumns() {
            if (this.columns.filter((ev) => ev.locked)) {
                const columns = [...this.columns];
                console.log(1);
                return columns
                    .sort((a, b) => Number(b.locked) - Number(a.locked))
                    .map((e) => {
                        return { ...e, orderIndex: undefined };
                    });
            } else {
                this.columns;
            }
            return null;
        },
    },
    methods: {
        filterChange: function (ev) {
            this.filter = ev.filter;
        },
        handleColumnReorder(event) {
            const lastLockedIndex = this.lockedColumns.findLastIndex((e) => e.locked);

            const updatedColumns = event.columns
                .sort((a, b) => Number(a.orderIndex) - Number(b.orderIndex))
                .map((e, index) => {
                    const newLockedState =
                        event.prev > lastLockedIndex && event.next <= lastLockedIndex
                            ? true
                            : event.prev <= lastLockedIndex && event.next > lastLockedIndex
                                ? false
                                : e.locked;
                    return {
                        ...e,
                        locked: index === event.next ? newLockedState : e.locked,
                    };
                });
            this.columns = updatedColumns;
        },
        isColumnLocked(columnName) {
            return this.columns.filter((ev) => ev.field === columnName)[0].locked;
        },
        handleFilterChange: function (filter) {
            this.result = process(products.slice(0), { filter: filter });
        },
        createDataState(dataState) {
            this.result = process(products.slice(0), dataState);
            this.dataState = dataState;
        },
        dataStateChange(event) {
            this.createDataState(event.data);
        },
        onColumnsSubmit(columnsState) {
            this.columns = columnsState;
        },
        lockChange(state, columnName) {
            const columnToLock = this.columns.filter((ev) => ev.field === columnName.field)[0];
            columnToLock.locked = state;
        },
        visibilityChange(state, columnName) {
            this.columns.filter((ev) => ev.field === columnName.field)[0].hidden = state;
        },
    },
};
</script>
  