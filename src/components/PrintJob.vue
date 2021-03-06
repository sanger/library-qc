<template>
  <div class="print-job">
    <alert ref='alert'></alert>
    <form method="post" action="#" v-on:submit.prevent="execute">
      <h4>Print a barcode</h4>
      <div class="form-group">
        <div class="row">
          <div class="col-md-3">
            <label for="printer">Select a Printer</label>
          </div>
          <div class="col-md-5">
            <select id="printer-list" class="form-control" name="printer-list" v-model="printerName">
              <option v-for="printerName in printerList" v-bind:key="printerName" v-bind:value="printerName">{{printerName}}</option>
            </select>
          </div>
          <div class="error">{{errors.printerName}}</div>
        </div>
      </div>
      <div class="form-group">
        <div class="row">
          <div class="col-md-3">
            <label for="barcode">Scan your plate barcode</label>
          </div>
          <div class="col-md-5">
            <textarea name="barcodes" id="barcodes" class="form-control" v-model="barcodes" rows="10" cols="10" />
          </div>
          <div class="error">{{errors.barcode}}</div>
        </div>
      </div>
      <div class="form-group">
        <div class="row">
          <div class="col-md-3"></div>
          <div class="col-md-5">
            <button name="submit" id="print" class="btn btn-success btn-block" type="submit">
              Print
            </button>
          </div>
        </div>
      </div>
    </form>
  </div>
</template>

<script>

import Model from '@/api/PrintMyBarcode'
import Alert from '@/components/Alert'
import PrinterList from '@/config/PrinterList'

export default {
  name: 'PrintJob',
  props: {
    labelTemplateId: {
      type: String,
      default: process.env.VUE_APP_LABEL_TEMPLATE_ID
    }
  },
  data () {
    return {
      msg: 'PrintJob',
      barcodes: '',
      printerName: PrinterList[0],
      date: new Date(),
      model: {},
      errors: {},
      printerList: PrinterList
    }
  },
  computed: {
    today () {
      return `${this.date.getDate().toString().padStart(2,'0')}-${this.months[this.date.getMonth()]}-${this.date.getFullYear()}`
    },
    labels () {
      let self = this
      return this.barcodes.split('\n').filter(Boolean).map(barcode => {
        return { 
          main_label: {
            top_left: self.today,
            bottom_left: barcode.concat('-QC'),  
            barcode: barcode.concat('-QC')
          } 
        }
      })
    },
    attributes () {
      return {
        labelTemplateId: this.labelTemplateId,
        printerName: this.printerName,
        labels: {
          body: this.labels
        }
      }
    },
    months () {
      return ['JAN','FEB','MAR','APR','MAY','JUN','JUL','AUG','SEP','OCT','NOV','DEC'] 
    }
  },
  components: {
    Alert
  },
  methods: {
    execute () {
      if (this.valid()) {
        this.model = new Model(this.attributes)
        this.model.save().then(success => {
          if(success) {
            this.$refs.alert.show('barcode successfully printed','success')
          } else {
            /*eslint no-console: ["error", { allow: ["error"] }] */
            console.error(this.model.errors)
            this.$refs.alert.show('barcode printing failed','danger')
          }
        })
      }
    },
    valid () {
      this.errors = {}
      if(!this.barcodes) {
        this.errors['barcodes'] = 'must be completed'
      }
      if(!this.printerName) {
        this.errors['printerName'] = 'must be completed'
      }
      return Object.keys(this.errors).length === 0
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
  form {
    width: 80%;
    margin: 10% auto;
  }
  .error {
    color: red;
  }
</style>