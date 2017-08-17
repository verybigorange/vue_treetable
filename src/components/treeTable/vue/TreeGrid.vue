<template>
  <el-table :data="data" border style="width: 100%" :row-style="showTr">
    <!--  column每个字段对象 index为每列  -->
    <el-table-column v-for="(column, index) in columns" :key="column.dataIndex" :label="column.text">
      <template scope="scope">
        <span v-if="spaceIconShow(index)" v-for="(space, levelIndex) in scope.row._level" class="ms-tree-space"></span>
        <!-- 切换树按钮 通过是否有子和列数来决定是否出现  -->
        <button style="padding:2px 3px;" class="button is-outlined is-primary is-small" v-if="toggleIconShow(index,scope.row)" @click="toggle(scope.$index)">
          <!-- 按钮的每种状态根据行展开数据来决定  -->
          <i v-if="scope.row._expanded" class="el-icon-caret-right" aria-hidden="true"></i>
          <i v-if="!scope.row._expanded" class="el-icon-caret-bottom" aria-hidden="true"></i>
        </button>
        <span v-else-if="index===0" class="ms-tree-space"></span>
        {{scope.row[column.dataIndex]}}
      </template>
    </el-table-column>
    <el-table-column label="操作" v-if="treeType === 'normal'" width="260">
      <template scope="scope">
  
        <!-- 点击按钮，将按钮的当前行数据和当前行的索引，以及数组和全局对象传过去  -->
        <el-button v-for="(item,i) in buttons" :key="i" size="small" :type="item.type" @click="item.event(scope.row,scope.$index,scope.store.states.data,scope)">
          {{item.text}}
        </el-button>
  
      </template>
    </el-table-column>
  </el-table>
</template>
<script>

import Utils from '../utils/index.js'
import vue from "vue"
import axios from "axios";

export default {
  name: 'tree-grid',
  props: {
    buttons: {
      type: Array,
      default: function () {
        return []
      }
    },
    // 该属性是确认父组件传过来的数据是否已经是树形结构了，如果是，则不需要进行树形格式化
    treeStructure: {
      type: Boolean,
      default: function () {
        return false
      }
    },
    // 这是相应的字段展示
    columns: {
      type: Array,
      default: function () {
        return []
      }
    },
    // 这是数据源
    dataSource: {
      type: Array,
      default: function () {
        return []
      }
    },
    // 这个作用是根据自己需求来的，比如在操作中涉及相关按钮编辑，删除等，需要向服务端发送请求，则可以把url传过来
    requestUrl: {
      type: String,
      default: function () {
        return ''
      }
    },
    // 这个是是否展示操作列
    treeType: {
      type: String,
      default: function () {
        return 'normal'
      }
    },
    // 是否默认展开所有树
    defaultExpandAll: {
      type: Boolean,
      default: function () {
        return true
      }
    }
  },
  computed: {
    // 格式化数据源
    data: function () {
      let me = this
      if (me.treeStructure) {
        let data = [];

        data = Utils.MSDataTransfer.treeToArray(me.dataSource, null, null, me.defaultExpandAll)

        return data
      }
      return me.dataSource
    }
  },
  methods: {
    // 显示行
    showTr: function (row, index) {
      let show = (row._parent ? (!row._parent._expanded && row._parent._show) : true)
      row._show = show
      return show ? '' : 'display:none;'
    },
    // 展开下级
    async toggle(trIndex) {
      
      let me = this
      let record = me.data[trIndex]   //拿到被点击当前行的数据

      record._expanded = !record._expanded

      //展开的时候才去请求新数据
      if(!record._expanded){
          // 测试用的请求
        let res = await axios.post('/api' + record.url, { "id": record.id });

        //调用设置新数据的函数。
        this.setNewData(this, res.data, trIndex, record._level)
      }
    },
    // 显示层级关系的空格和图标
    spaceIconShow(index) {
      let me = this
      if (me.treeStructure && index === 0) {
        return true
      }
      return false
    },
    // 点击展开和关闭的时候，图标的切换
    toggleIconShow(index, record) {

      let me = this
      if (record.isHasChildren && index === 0) {
        return true
      }
      return false
    },
    // 递归寻找需要删除的子元素长度
    DelLen(currentTrData) {
      var len = 0;
      //递归遍历需要删除的子元素长度
      (function changeDelLen(currentTrData) {
        if (currentTrData.children && currentTrData.children.length > 0) {
          len += currentTrData.children.length;
          for (var i in currentTrData.children) {
            if (currentTrData.children[i].hasOwnProperty("children")) {
              changeDelLen(currentTrData.children[i]);
            }
          }
        }
      })(currentTrData)
      return len;
    },
    //参数1上下文this；参数2点击请求的新子数据；参数3为当前行的索引；参数4当前行所属树层级
    setNewData(context, newdata, trIndex, level) {
      //转换新数据
      var datahandled = Utils.MSDataTransfer.treeToArray(newdata, context.data[trIndex], level, context.defaultExpandAll);

      var len = this.DelLen(context.data[trIndex]);

      // 如果当前点击行有子的话先删除子。
      if (len > 0) context.data.splice(trIndex + 1, len);

      //把新的子挂载到当前点击行
      context.data[trIndex].children = datahandled

      //追加新子值到数组中去
      for (var i = (datahandled.length - 1); i >= 0; i--) {
        context.data.splice(trIndex + 1, 0, datahandled[i])
      }
    },
  }
}
</script>
<style scoped>
.ms-tree-space {
  position: relative;
  top: 1px;
  display: inline-block;
  font-family: 'Glyphicons Halflings';
  font-style: normal;
  font-weight: 400;
  line-height: 1;
  width: 30px;
  height: 14px;
}

.ms-tree-space::before {
  content: ""
}

table td {
  line-height: 26px;
}
</style>
