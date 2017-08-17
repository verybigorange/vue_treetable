<template>
  <div class="hello">
    <tree-grid :columns="columns" :tree-structure="true" :data-source="dataSource"></tree-grid>
  </div>
</template>

<script>
import {TreeGrid} from './treeTable'
import axios from 'axios';


export default {
  beforeMount(){
    axios.post("/api/data").then(data=>{
      this.dataSource = data.data;
     
    })
  },
  name: 'hello',
  data () {
    return {
      columns: [
          {
            text: '城市',
            dataIndex: 'name'
          },
          {
            text: '人口（万）',
            dataIndex: 'people'
          },
          {
            text: '温度(℃)',
            dataIndex: 'temperature'
          }
        ],
        dataSource: [],
    }
  },
  components: {
    TreeGrid
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
