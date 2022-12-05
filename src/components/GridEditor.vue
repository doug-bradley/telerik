<template>
  <k-dialog :width="400"
                :height="450"
                @close="cancel">
    <div class="k-form k-form-horizontal">

      <!-- 

        field: "InvoiceRef",
        title: "Ref",
        
        field: "InvoiceDate",
        filter: "date",
        
        field: "PostingDate",
        filter: "date",
        
        field: "SupplierName", 
        title: "Vendor"
        
        field: "ProcessorName", 
        title: "Financial Assistant"
        
        field: "CurrentStateName",
        title: "Current State"
        
       -->
      <div class="k-form-field">
        <kendo-label for="Ref" class="k-form-label">Invoice #</kendo-label>
        <div class="k-form-field-wrap">
          <k-input :style="{ width: '230px' }" type="text" :name="'InvoiceRef'"
            v-model="productInEdit.InvoiceRef"></k-input>
        </div>
      </div>
      
      
      <div class="k-form-field">
        <kendo-label for="Ref" class="k-form-label">Invoice Date</kendo-label>
        <div class="k-form-field-wrap">
          <date-picker :style="{ width: '230px' }" :default-value="new Date()" :default-Show="true"/>
        </div>
      </div>
            
      
      <div class="k-form-field">
        <kendo-label for="Ref" class="k-form-label">Posting Date</kendo-label>
        <div class="k-form-field-wrap">
          <date-picker :style="{ width: '230px' }" :default-value="new Date()" :default-Show="true"/>
        </div>
      </div>
      
      
      <div class="k-form-field">
        <kendo-label for="Ref" class="k-form-label">Supplier</kendo-label>
        <div class="k-form-field-wrap">
          
          
          <drop-down-list
            :style="{ width: '230px' }" 
            :data-items='suppliers'
            :default-item="'Select Supplier ...'"
        />

        
        </div>
      </div>

      <div class="k-form-field">
        <kendo-label for="Ref" class="k-form-label">Company</kendo-label>
        <div class="k-form-field-wrap">
          
          <drop-down-list
            :style="{ width: '230px' }" 
            :data-items='companies'
            :default-item="'Select Processor ...'"
        />

        </div>
      </div>
<!-- 
      <div class="k-form-field">
        <kendo-label for="Ref" class="k-form-label">Processor</kendo-label>
        <div class="k-form-field-wrap">
          
          <drop-down-list
            :style="{ width: '230px' }" 
            :data-items='processors'
            :default-item="'Select Processor ...'"
        />

        </div>
      </div>


      <div class="k-form-field">
        <kendo-label for="Ref" class="k-form-label">State</kendo-label>
        <div class="k-form-field-wrap">
          
          
          <drop-down-list
            :style="{ width: '230px' }" 
            :data-items='states'
            :default-item="'Select State ...'"
        />

        
        </div>
      </div> -->



    </div>

    <dialog-actions-bar>
      <kendo-button @click="cancel"> Cancel </kendo-button>
      <kendo-button :theme-color="'primary'" @click="save"> Save </kendo-button>
    </dialog-actions-bar>
  </k-dialog>
</template>


<script>
import { Dialog, DialogActionsBar } from '@progress/kendo-vue-dialogs';
import { Input } from '@progress/kendo-vue-inputs';
import { Label } from '@progress/kendo-vue-labels';
import { Button } from '@progress/kendo-vue-buttons';
import { DatePicker } from '@progress/kendo-vue-dateinputs';
import { DropDownList } from '@progress/kendo-vue-dropdowns';

export default {
  components: {
    'k-input': Input,
    'k-dialog': Dialog,
    'dialog-actions-bar': DialogActionsBar,
    'kendo-button': Button,
    'kendo-label': Label,
    'date-picker': DatePicker,
    'drop-down-list': DropDownList
  },
  props: {
    dataItem: Object,
  },
  data: function () {
    return {
      productInEdit: {
        ProductName: '',
        UnitsInStock: 0,
        Discontinued: false,
      },
      suppliers: ['Supplier 1', 'Supplier 2', 'Supplier 3'], 
      processors: ['John Doe', 'Jane Doe', 'John Smith'],
      states: ['New', 'In Progress', 'Completed'],
      companies: ['Amazon', 'Google', 'Microsoft']
    };
  },
  created: function () {
    if (this.$props.dataItem) {
      this.productInEdit = this.$props.dataItem;
    }
  },
  methods: {
    cancel() {
      this.$emit('cancel');
    },
    save() {
      this.$emit('save');
    },
  },
};
</script>
  
<style>
.k-form-label {
  white-space: nowrap;
}

</style> 