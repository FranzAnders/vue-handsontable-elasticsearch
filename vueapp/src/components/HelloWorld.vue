<template>
  <div id="example-container" class="wrapper">
    <div class="controller" ><v-btn class="save-btn" dark @click="saveData()" >Save</v-btn></div>
    <div id="hot-preview" v-if="!!hotSettings.data" >
      <HotTable ref="hottable" :root="root" :settings="hotSettings" @myAfterPaste="doSomething"></HotTable>
    </div>
    <div class="text-xs-center pagination-bar">
      <v-pagination v-if="!!hotSettings.data" :length="Math.ceil(totalRows/rowsPerPage)" v-model="page" :total-visible="10" @next="nextPage" @previous="prevPage" @input="selectPage" ></v-pagination>
      <div class="pagination-info" ><input type="number" v-model="rowsPerPage" @change="changeRowsPerPage" />&nbsp;/&nbsp;page</div>
    </div>
    <div v-if="saving" >Saving.....</div>
  </div>
</template>

<script>
/* eslint-disable */
  import HotTable from './vue-handsontable-official';
  import Vue from 'vue';
  import Vuetify from 'vuetify'
  import axios from 'axios'
  import VueAxios from 'vue-axios'
  import { HomeUrl } from './config'
 
  Vue.use(Vuetify)
  Vue.use(VueAxios, axios)

  export default {
    data: function() {
      const w = Math.max(document.documentElement.clientWidth, window.innerWidth || 0);
      const h = Math.max(document.documentElement.clientHeight, window.innerHeight || 0);
      return {
        rendering: false,
        page: 1,
        totalRows: 0,
        rowsPerPage: 5,
        saving: false,
        changedElments: [],
        root: 'test-hot',
        hotTableKey: 'helloworld',
        ids: [],
        hotSettings: {
          data: undefined,
          width: w * 0.8,
          height: h * 0.7,
          colHeaders: [
            'Id',
            'No',
            'Name',
            'Alt Names',
            'Latitude',
            'Longitude',
            'Country',
            'Population',
          ],
          columns: [
            {
              data: 'id',
              editor: 'text'
            },
            {
              data: 'no',
              readOnly: true,
            },
            {
              data: 'name',
              editor: 'text'
            },
            {
              data: 'alt_names',
              editor: 'text'
            },
            {
              data: 'latitude',
              editor: 'text'
            },
            {
              data: 'longitude',
              editor: 'text'
            },
            {
              data: 'country_code',
              editor: 'text'
            },
            {
              data: 'population',
              editor: 'text'
            }
          ],
          maxRows: 1000,
          rowHeaders: true,
          columnSorting: true,
          colWidths: [80, 150, 400, 150, 150, 150, 150, 150],
          autoColumnSize: {
            samplingRatio: 23
          },
          manualColumnResize: false,
          afterChange: function(changes, source){
            this.rootElement.__vue__.$emit('myAfterPaste', changes);
          }
        }
      };
    },
    created() {
      this.getDataFromAPI()
    },
    methods: {
      doSomething: function (changes, source) {
        if (changes)
        changes.forEach( (change, index) => {
          const obj = []
          console.log('change:', change)
          obj.id = this.$refs.hottable.table.getDataAtCell(change[0], 0)
          obj.prop = change[1]
          obj.value = change[3]
          this.changedElments.push(obj)
        })
        
      },

      saveData() {
        const data = this.changedElments
        if (data.length === 0) return
        // const testUrl = 'http://localhost:9200'
        const api = `${HomeUrl}/web/web`
        let responseCount = data.length
        this.saving = true
        const comp = this
        
        data.forEach( (element, index) => {
          const property = element.prop
          const obj =  {};
          obj[property] = element.value
          axios.post(`${api}/${element.id}/_update`, {
            "doc": obj
          })
          .then(function (response) {
          
            responseCount --
            if (responseCount === 0) {
              comp.saving = false
              comp.changedElments = []
            }

          })
          .catch(function (error) {
            console.log(error);
          });
        });
      },

      getDataFromAPI() {
        const api = `${HomeUrl}/web/_search`
        const cmp = this
        const query = {
          query: { 'match_all': {} },
          sort: { '_id': { 'order': 'asc'} },
          size: this.rowsPerPage,
          from: (this.page - 1) * this.rowsPerPage
        }
        axios.get(api, {
          params: {
            source: JSON.stringify(query),
            source_content_type: 'application/json'
          }
        })
        .then(response => {
         const obj = response.data.hits.hits
          this.totalRows = response.data.hits.total
          const origin = (this.page-1)*this.rowsPerPage
          const data = []
          obj.forEach((element, index) => {
            // cmp.ids.push(element._id)
            data.push({
              id: element._id,
              no: origin+index+1,
              name: element._source.name,
              alt_names: element._source.alt_names,
              latitude: element._source.latitude,
              longitude: element._source.longitude,
              country_code: element._source.country_code,
              population: element._source.population
            })
          });
          this.hotSettings.data = data;
        })
        .catch(e => {
          this.errors.push(e)
        })
      },
       nextPage() {
         this.getDataFromAPI()
       },
       prevPage() {
         this.getDataFromAPI()
       },
       selectPage() {
          this.getDataFromAPI()
       },
       changeRowsPerPage() {
         this.getDataFromAPI()
       }
    },
    name: 'HelloWorld',
    components: {
      HotTable
    }
  }
</script>

<style>
  #example-container {
    position: relative;
  }
  #test-hot {
    margin: auto;
    border: solid 0.7px lightgray;
  }
  table tbody tr td:first-of-type {
    display: none;
  }
  table thead tr th:first-of-type {
    border-right: 1px solid #eaeaea !important;
  }
  table thead tr th:nth-of-type(2) {
    display: none;
  }
  #hot-options {
    width: 200px;
    position: absolute;
    top: 0;
    left: 0;
    padding: 15px;
    font-size: 14px;
  }

  table th, table td {
    min-width: 100px;
    max-width: 300px;
    word-wrap: break-word;
  }
  .handsontableInput {
    max-width: 300px !important;
  }
  .handsontable td {
    padding: 5px !important;
  }

  .handsontable th {
    vertical-align: middle !important;
  }

  .handsontable .columnSorting {
    position: relative;
    display: inline-block;
    width: 100%;
    height: 100%;
    cursor: pointer;
  }

  .controller {
    width: 80vw;
    text-align: right;
    margin: auto;
    padding: 20px;
    margin-top: -10vh;
    box-sizing: border-box;
  }
  .save-btn {
    font-size: 1em;
    border-radius: 10px;
    padding: 0.2em 1.5em;
    cursor: pointer;
    outline: none;
  }

  .save-btn:hover {
    background-color: aliceblue;
  }
  .save-btn:active {
    color: darkgray;
  }

  .pagination-bar {
    position: relative;
  }
  .pagination__item {
    outline: none;
  }
  .pagination__item--active {
    border: solid 2px orange;
  }
  .pagination-info {
    position: absolute;
    top: 5px;
    left: 10vw;
  }
  .pagination-info input {
    width: 60px;
    border: SOLID 1px lightgray;
    border-radius: 5px;
    padding: 5px 0 5px 10px;
    color: #222;
  }
</style>