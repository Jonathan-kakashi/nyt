<template>
  <CCard>
    <CCardHeader class="pointer" @click="line1()">
      <slot name="header" >
        <CIcon name="cil-grid"/>Line No:1  {{description}}
      </slot>
    </CCardHeader>    
    <CCardBody>
      <CCard>
        <table style="width:100%">
          <tr >
            <th style="border-bottom:1px solid #dddddd;border-right:1px solid #dddddd;text-align:center;">Target Total</th>
            <th style="border-right:1px solid #dddddd;text-align:center;">Actual Total</th>
            <th style="border-bottom:1px solid #dddddd;text-align:center;">CAP(%)</th>
          </tr>
          <tr>
            <td style="text-align:center;">{{TotalTarget}}</td>
            <td style="border-right: 1px solid #dddddd;border-top: 1px solid #dddddd;border-left: 1px solid #dddddd;text-align:center;">{{actualTotal}}</td>
            <td style="text-align:center;">90%</td>
          </tr>
        </table>
      </CCard>
      <div style="display:block">
        AVG EFF % :<input :value=effective style="height:20px;border:none" />
        <b v-if="browstatus" style="float:right">BR:Y</b>
        <b v-else style="float:right">BR:N</b>
      </div>
      <CProgress
        :color="progress_color"             
        class="mb-2"
        v-model="effective"
      />
      
      <CDataTable
        :hover="hover"
        :striped="striped"        
        :small="small"
        :fixed="fixed"
        :items="items"
        :fields="fields"
        :dark="dark"
        :spacing="spacing"
         border
        style="text-align:center"
        sorter     
      >
        <template #Target="{item}">
          <td style="text-align:center;">
            {{item.realtime_target}}
          </td>
        </template>
        <template #EFF="{item}">
          <td style="text-align:center">
            {{item.EFF}}
          </td>
        </template>
        <template #Actual="{item}">
          <td style="text-align:center">
            {{item.Actual}}
          </td>
        </template>
        <template #NP="{item}">
          <td style="text-align:center">
            {{item.NP}}
          </td>
        </template>
        <template #Machine="{item}" >
          <td style="text-align:center">
            {{item.machine_id}}
          </td>
        </template>
      </CDataTable>
    </CCardBody>
  </CCard>
</template>
<style scoped>
.pointer{
  cursor: pointer;
}
.card-body{
  padding: 5px!important;
}
.table-bordered td{
  padding:0px!important;
}

/* .table_border{

} */
.flashit{
  color:#f2f;
	-webkit-animation: flash linear 1s infinite;
	animation: flash linear 1s infinite;
}
@-webkit-keyframes flash {
	0% { opacity: 1; } 
	50% { opacity: .1; } 
	100% { opacity: 1; }
}
@keyframes flash {
	0% { opacity: 1; } 
	50% { opacity: .1; } 
	100% { opacity: 1; }
}
</style>
<script>
import Vue from 'vue'
import axios from 'axios'
import VueAxios from 'vue-axios'
"use strict";

Vue.use(VueAxios, axios)
export default {
  name: 'Table',
  props: {
    items: Array,
    spacing:2,
    fields: {
      type: Array,
      default () {
        return ['Machine', 'Target', 'Actual', 'EFF', 'NP']
      }
    },
    caption: {
      type: String,
      default: 'Line No:1'
    },
    hover: Boolean,
    striped: Boolean,
    bordered: Boolean,
    small: Boolean,
    fixed: Boolean,
    dark: Boolean,
  },
  data:function(){
    return{
      timer:'',
    browstatus:"",
    effective:0,
    description:"",
    progress:0,
    progress_color:'',
    TotalTarget:0,
    CAP:"",
    SelNPP:"",
    actualTotal:"",
    DataList1:[]
    }
  },
  created(){
  this.appData()
  this.interval()
  this.BrowStatus()
  },
  methods: {
    BrowStatus(){
      if(localStorage.getItem('BR_D_Line1') == '' || localStorage.getItem('BR_D_Line1') == null){
        this.browstatus = true
      }else if(localStorage.getItem('BR_D_Line1') == 'BR_Sai5_01'){
        this.browstatus = false
      }
    },
  interval(){
    setInterval(() => {
		 this.appData()
	}, 300000)
  },    
  async appData(){
    var response = await Vue.axios.get('http://nyiot.nanyangtextile.com/nyiot/carding_feed/get_card_nyt_sai5.php');
    var DataList = response.data;
    this.DataList1 = []
    var getTarget = 0
    var getActual = 0
    var getdescription = ""
    var geteffective = 0
    for (var i=0;i<DataList.length;i++) {
      if(DataList[i].PD_Line == '1'){        
        DataList[i]['EFF']="";
        DataList[i]['hour']="";
        DataList[i]['min']="";
        DataList[i]['second']="";
        DataList[i]['Actual']="";
        DataList[i]['realtime_target']="";
        DataList[i]['totalSec']="";
        if(this.DataList1.length < 6){
          if(DataList[i].log_time.substring(11, 13) >= 0 && DataList[i].log_time.substring(11, 13) < 8){
            DataList[i].hour = (Number(DataList[i].log_time.substring(11, 13))+Number(16))*3600;
          }else{
            DataList[i].hour = (DataList[i].log_time.substring(11, 13)-8)*3600;
          }
          DataList[i].min = DataList[i].log_time.substring(14, 16)*60;
          DataList[i].second = DataList[i].log_time.substring(17, 19)*1;
          DataList[i].totalSec = DataList[i].hour+DataList[i].min+DataList[i].second;
          DataList[i].realtime_target = ((DataList[i].target_weight*DataList[i].gram_cm*90*DataList[i].totalSec)/(60*1000)).toFixed(2);          
          DataList[i].Actual = (DataList[i].weight_gram/1000).toFixed(2)
          DataList[i].EFF = ((DataList[i].Actual*100)/DataList[i].realtime_target)
          getActual = getActual + Number(DataList[i].Actual)
          getTarget =getTarget + Number(DataList[i].realtime_target)
          geteffective = geteffective + Number(DataList[i].EFF)
          getdescription = DataList[i].yarn_type
          this.DataList1.push(DataList[i])
        }  
      }
    }
    this.TotalTarget = new Intl.NumberFormat().format(getTarget.toFixed(2))
    this.actualTotal = new Intl.NumberFormat().format(getActual.toFixed(2))
    this.CAP = (this.TotalTarget*100/this.actualTotal).toFixed(2)
    this.description = getdescription
    
    let apiCounts_String = localStorage.getItem('api_counts_line1');
    let apiCounts = 0;
    if(apiCounts != null) {
      apiCounts = parseInt(apiCounts_String);
    }
    let dd = localStorage.getItem('initial_progress_line1');
    if(dd == null || apiCounts%60==0) {
      apiCounts = 0;
      dd= (geteffective/this.DataList1.length).toFixed(2);
      localStorage.setItem('initial_progress_line1', Number((geteffective/this.DataList1.length).toFixed(2))); 
    }
    localStorage.setItem('api_counts_line1', apiCounts++);
    this.effective = Number(dd);
    localStorage.setItem('progress_linedetail_line1', this.effective);


    if(this.effective > 90){
      this.progress_color = "success"
    }else if(this.effective > 70 && this.effective <= 90){
      this.progress_color = "warning"
    }else if(this.effective <= 70){
      this.progress_color = "danger"
    }
    },
    getBadge (status) {
      return status === 'Active' ? 'success'
        : status === 'Inactive' ? 'secondary'
          : status === 'Pending' ? 'warning'
            : status === 'Banned' ? 'danger' : 'primary'
    },
    line1(){
      this.$router.push("/linedetail1")
    },
  }
}
</script>
