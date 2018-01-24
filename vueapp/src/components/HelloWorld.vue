<template>
  <div id="example-container" class="wrapper">
    <div id="hot-preview" v-if="rendering === true" >
      <HotTable ref="hottable" :root="root" :settings="hotSettings" ></HotTable>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
  import HotTable from './vue-handsontable-official';
  import Vue from 'vue';
  import axios from 'axios'
  import VueAxios from 'vue-axios'
  import { HomeUrl } from './config'

  Vue.use(VueAxios, axios)

  export default {
    data: function() {
      return {
        rendering: false,
        root: 'test-hot',
        hotTableKey: 'helloworld',
        hotSettings: {
          data: undefined,
          colHeaders: true,
          rowHeaders: true,
          columnSorting: true,
          manualColumnResize: true
        }
      };
    },
    async created() {
      this.rendering = false;
      await this.getDataFromAPI()
      
    },
    mounted: function() {
      setTimeout(this.rendering = true, 4000);
    },
    methods: {
      getHandsontableData() {
        const data = this.$refs.hottable.table.getData()
      },
      getDataFromAPI() {
        const api = `${HomeUrl}/web/_search`
        console.log('api', api)
        axios.get(api)
        .then(response => {
          const obj = response.data.hits.hits
          const data = []
      
          obj.forEach(element => {
            data.push([element._id,element._source.alt_names,
            element._source.name,
            element._source.latitude,
            element._source.longitude,element._source.country_code,element._source.population
            ])
          });
          console.log(data)
          this.hotSettings.data = data;
        })
        .catch(e => {
          this.errors.push(e)
        })
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

  #hot-options {
    width: 200px;
    position: absolute;
    top: 0;
    left: 0;
    padding: 15px;
    font-size: 14px;
  }

  #hot-preview {
  }

  #test-hot {
    max-width: 80vw;
    height: 70vh;
    margin: auto;
    overflow: scroll;
    width: fit-content;
  }

  .ht_master.handsontable {
    width: fit-content;
    margin: auto;
  }

  .ht_master .wtHolder {
    width: fit-content !important;
  }

  table th, table td {
    min-width: 100px;
    max-width: 300px;
    max-height:300px;
    overflow-y: scroll;
  }

  .handsontable .columnSorting {
    position: relative;
    display: inline-block;
    width: 100%;
    height: 100%;
    cursor: pointer;
  }
</style>