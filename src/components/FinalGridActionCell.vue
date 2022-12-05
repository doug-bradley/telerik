<template>
    <td class="k-command-cell">
        <kendo-button class="k-grid-edit-command" @click="editHandler">
            Edit
        </kendo-button>

        <kendo-button :theme-color="'primary'" class="k-grid-remove-command" @click="removeHandler">
            Delete
        </kendo-button>
    </td>
</template>
<script>
import { Button } from '@progress/kendo-vue-buttons';

export default {
    components: {
        'kendoButton': Button
    },
    props: {
        field: String,
        dataItem: Object,
        format: String,
        className: String,
        columnIndex: Number,
        columnsCount: Number,
        rowType: String,
        level: Number,
        expanded: Boolean,
        editor: String
    },
    methods: {
        editHandler: function () {
            this.$emit('edit', this.dataItem);
        },
        removeHandler: function () {
            if (confirm('Confirm deleting: ' + this.dataItem.ProductName)) {
                this.$emit('remove', this.dataItem);
            }
        }
    }
}
</script>
<style>
.k-button.k-grid-edit-command {
    background-color: #008897;
    color: white;
}
</style>