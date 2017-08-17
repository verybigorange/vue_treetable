<template>
  <div class="hello">
    <tree-grid :columns="columns" :tree-structure="true" :data-source="dataSource" :buttons="buttons"></tree-grid>
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

      // 列字段
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
        // 操作按钮
        buttons:[
          {
            text:"删除",
            type:"danger",
            event:function(currentTrData,index,dataArr,scope){
               //获取需要删除的子元素个数 
              let len = scope._self.DelLen(currentTrData);
              dataArr.splice(index,1+len);
            }
          },{
            text:"编辑",
            type:"default",
            event:function(currentTrData,index,data){}
          },{
            text:"添加下级树结构",
            type:"success",
            event:function(currentTrData,index,data){}
          }
        ]
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
