<template>
  <div id="example-container" class="wrapper">
    <div id="hot-preview">
      <HotTable :root="root" :settings="hotSettings"></HotTable>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
  import HotTable from './vue-handsontable-official';
  import Vue from 'vue';

  export default {
    data: function() {
      return {
        root: 'test-hot',
        hotSettings: {
          data: function(rows, columns) {
            const result = [];
            for (let i = 0; i < rows; i++) {
              let row = [];
              for (let j = 0; j < columns; j++) {
                row.push('[' + i + ', ' + j + ']');
              }
              result.push(row);
            }
            return result;
          }(1000, 7),
          colHeaders: true,
          rowHeaders: true,
          columnSorting: true,
          manualColumnResize: true
        }
      };
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
    overflow: hidden;
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
  }

  .handsontable .columnSorting {
    position: relative;
    display: inline-block;
    width: 100%;
    height: 100%;
    cursor: pointer;
  }
</style>