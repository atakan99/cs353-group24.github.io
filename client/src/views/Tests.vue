<template>
  <v-app>
    <v-container class="">
      <v-row>
        <h1 class="ml-5 mt-10 pt-5 datatablefontcolor--text">Tests</h1>
      </v-row>
      <PaginationTable :items="items" :headers="headers" :tableInfo="tableInfo" :buttonHeader="buttonHeader" style="margin-top:1.5rem" class="mx-2">
          <template #buttons="{item}">
            <v-row>
            <v-col class="d-flex justify-center mx-n8">
                <v-btn :disabled="(item.status !== 'finalized')"  @click="handleDialog1(item)" class="rounded-lg font-weight-bold rounded-pill" outlined color="#5080DE">See Components</v-btn>
            </v-col>
            </v-row>
          </template>
      </PaginationTable>
      <v-snackbar
          v-model="snackbar"
          :timeout="5000"
      >
        {{ errorMsg }}

        <template v-slot:action="{ attrs }">
          <v-btn
              color="indigo"
              text
              v-bind="attrs"
              @click="snackbar = false"
          >
            Close
          </v-btn>
        </template>
      </v-snackbar>
      <v-overlay :value="overlay">
        <v-progress-circular
            indeterminate
            size="64"
        ></v-progress-circular>
      </v-overlay>
      <Dialog :tableData="group" :dialog="dialog1" :title="item.calories" @close="dialog1=false">
        <template #tableActions="{item}">
            <v-btn @click="handleDialog2(item)" class="rounded-pill font-weight-bold" outlined color="#5080DE">History</v-btn>
        </template>
      </Dialog>
      <Dialog :subtitle="subtitle" :tableData="group1" back @back="handleBack" :dialog="dialog2" :title="item1.name" @close="dialog2=false"></Dialog>
    </v-container>
  </v-app>
</template>

<script>
import PaginationTable from "@/components/PaginationTable";
import Dialog from "@/components/Dialog"
export default {
  components: {
    PaginationTable, Dialog
  },

  data: () => ({
    snackbar: false,
    overlay: false,
    errorMsg: '',
    id:'',
    item:{},
    item1:{},
    group: {
        items:'',
        headers:'',
        tableInfo:'',
        buttonHeader: ''
    },
    group1: {
      items:'',
      headers:'',
      tableInfo:'',
      buttonHeader: ''
    },
    dialog1:false,
    dialog2:false,
    buttonHeader: 'details',
    headers: [
    {
        text: 'Appointment ID',
        align: 'start',
        // sortable: false,
        // filterable: false,
        value: 'id',
        class: 'datatablefontcolor--text'
    },
    { text: 'Test Name', value: 'name', class: 'datatablefontcolor--text' },
    { text: 'Date', value: 'date', class: 'datatablefontcolor--text' },
    { text: 'Status', value: 'status', class: 'datatablefontcolor--text' },
    { text: 'Details', value: 'details', sortable:false, class: 'datatablefontcolor--text' },
    ],
    subtitle: '',
    items: [],
    tableInfo: {
        tableTitle: 'Tests',
        itemsKey: 'name',
        itemsPerPage: 6,
    },
  }),
  methods: {
    async getItems(){
      this.overlay = true
      if(this.$cookies.get('user'))
      {
        this.id = this.$cookies.get('user').national_id
        let temp = this.$cookies.get('user').name
        this.patientName = temp.charAt(0).toUpperCase() + temp.slice(1)
      }
      await this.$http.get(this.$url+`/patient/${this.id}/see_all_tests`).then(res => {
        this.items = []
        res.data.forEach(x => {
          let temp = {
            id: x.appointment_id,
            name: x.test_name,
            date: x.date_to_char,
            status: x.test_status,
            resultId: x.result_id
          }
          this.items.push(temp)
        })
        this.overlay = false
      }).catch((err) => {
        console.log(err)
        this.errorMsg = 'Unexpected Error, could not load data'
        this.overlay = false
        this.snackbar = true
      })
    },
        handleBack: function(){
            this.dialog2 = false;
            this.dialog1 = true;
        },
        async handleDialog1(item){
          let componentItems = [];
          this.overlay = true
          await this.$http.get(this.$url+`/patient/${this.id}/see_test_comps`,{
            params:{
              result_id: item.resultId
            }
              }
          ).then(res => {
            res.data.forEach(x => {
              let temp = {
                id: x.result_id,
                name: x.comp_name,
                value: (x.comp_value == null) ? '-' : x.comp_value,
                result: (x.comp_result == null) ? '-' : x.comp_result,
                status: (x.comp_status == null) ? '-' : x.comp_status,
                higher: (x.upper_normality_interval == null) ? '-' : x.upper_normality_interval ,
                lower: (x.lower_normality_interval == null) ? '-' : x.lower_normality_interval,
              }
              componentItems.push(temp)
            })
            this.overlay = false
          }).catch((err) => {
            console.log(err)
            this.errorMsg = 'Unexpected Error, could not load data'
            this.overlay = false
            this.snackbar = true
          })
            this.item = item;
            this.dialog1 = true;
            this.group.items = componentItems;
            this.group.headers = [
              {
                text: 'Result ID',
                align: 'start',
                // sortable: false,
                // filterable: false,
                value: 'id',
                class: 'datatablefontcolor--text'
              },
              { text: 'Component Name', value: 'name', class: 'datatablefontcolor--text' },
              { text: 'Lower Normality Level', value: 'lower', class: 'datatablefontcolor--text' },
              { text: 'Upper Normality Level', value: 'higher', class: 'datatablefontcolor--text' },
              { text: 'Component Value', value: 'value', class: 'datatablefontcolor--text' },
              { text: 'Component Result', value: 'result', class: 'datatablefontcolor--text' },
              { text: 'Component Status', value: 'status', class: 'datatablefontcolor--text' },
              { text: 'History', value: 'History', sortable:false, class: 'datatablefontcolor--text' }
            ];
            this.group.buttonHeader = 'History'
        },
        async handleDialog2(item){
          this.item1 = item;
          let prevValues = [];
          this.overlay = true
          await this.$http.get(this.$url+`/patient/${this.id}/see_prev_test_comps`,{
                params:{
                  comp_name: item.name.toLo
                }
              }
          ).then(res => {
            console.log(res)
            res.data.forEach(x => {
              let temp = {
                date: (x.result_date == null) ? '-': x.result_date,
                aId: (x.appointment_id == null) ? '-' : x.appointment_id,
                rId: (x.result_id == null) ? '-' : x.result_id,
                value: (x.comp_value == null) ? '-' : x.comp_value,
                result: (x.comp_result == null)? '-' : x.comp_result,
                higher: (x.upper_normality_interval == null) ? '-' : x.upper_normality_interval ,
                lower: (x.lower_normality_interval == null) ? '-' : x.lower_normality_interval,
              }
              prevValues.push(temp)
              this.subtitle = 'Expected Interval: ' + String(temp.lower) + ' - ' + String(temp.higher)
            })
            this.overlay = false
          }).catch((err) => {
            console.log(err)
            this.errorMsg = 'Unexpected Error, could not load data'
            this.overlay = false
            this.snackbar = true
          })
          this.item = item;
          this.dialog1 = true;
          this.group1.items = prevValues;
          this.group1.headers = [
            {
              text: 'Date',
              align: 'start',
              // sortable: false,
              // filterable: false,
              value: 'date',
              class: 'datatablefontcolor--text'
            },
            { text: 'Appointment ID', value: 'aId', class: 'datatablefontcolor--text' },
            { text: 'Result ID', value: 'rId', class: 'datatablefontcolor--text' },
            { text: 'Lower Normality Level', value: 'lower', class: 'datatablefontcolor--text' },
            { text: 'Upper Normality Level', value: 'higher', class: 'datatablefontcolor--text' },
            { text: 'Component Value', value: 'value', class: 'datatablefontcolor--text' },
            { text: 'Component Result', value: 'result', class: 'datatablefontcolor--text' },
          ];
            this.dialog1 = false;
            this.dialog2 = true;
        },
    },
  mounted() {
    this.getItems()
  },
  created: function() {
        // this.group.items = this.items
        // this.group.headers = this.headers
        // this.group.tableInfo = this.tableInfo
        // this.group.buttonHeader = this.buttonHeader
    }
};
</script>

<style scoped>

</style>