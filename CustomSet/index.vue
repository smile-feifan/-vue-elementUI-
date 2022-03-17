@@ -0,0 +1,128 @@
<template>
 <div>
   <el-drawer
   :visible.sync="drawer_"
   :with-header="false"
   size="20%"
   >
     <div class="custom-setting">
       <h3>配置列项</h3>
       <el-button type="text" @click="handleReback">重置</el-button>
     </div>
     <el-tree 
     :key="colKey" 
     draggable 
     :data="col" 
     :props="defaultProps" 
     :allow-drag="allowDrag" 
     :allow-drop="allowDrop" 
     @node-drop="handleDrop"
     >
       <span class="tree-table-setting" slot-scope="{node,data}" :class="{fixedActivated: data.fixed !== null}">
         <span>
           <el-switch v-model="data.hidden" :active-value="false" :inactive-value="true"></el-switch>
           <span>{{node.label}}</span>
         </span>
         <span>
           <el-tooltip
            effect="dark" content="左固定" placement="top-start">
              <i id="left" :class="{activatedFixed: data.fixed === 'left'}" @click="handleFixed(data,'left')" class="iconfont icon-zuo-guding"></i>
           </el-tooltip>
           <el-tooltip
            effect="dark" content="右固定" placement="top-start">
              <i id="right" :class="{activatedFixed: data.fixed === 'right'}" @click="handleFixed(data,'right')" class="iconfont icon-you-guding"></i>
           </el-tooltip>
           <el-tooltip
            effect="dark" content="取消固定" placement="top-start">
              <i id="noFixed" :class="{activatedFixed: data.fixed === null}"  @click="cancelFixed(data)" class="iconfont icon-quxiaoguding"></i>
           </el-tooltip>
         </span>
       </span>
     </el-tree>
   </el-drawer>
 </div>
</template>

<script>
 export default {
   name:'CustomSet' ,
   props:{
     col:{
       type:Array,
       default:[]
     },
     drawer:{
       type:Boolean,
       default:false
     }
   },
   data () {
     return {
       colKey:1,
       //列设置中tree配置
       defaultProps: {
         children: 'children',
         label: 'label'
       },
     }
   },
   computed: {
     drawer_: {
       get() {
         return this.drawer
       },
       set(v) {
         this.$emit('changeDrawer',v)
       }
     }
   },
   methods:{
     //重置
      handleReback() {
        this.$emit('handleReback')
      },
      //是否允许拖拽
      allowDrag(draggingNode) {
        return draggingNode.data.fixed === null
      },
      //仅允许tree节点上下拖动
      allowDrop(draggingNode,dropNode,type) {
        return type !== 'inner'
      },
      // tree更新时拖动表格
      handleDrop() {
        this.$emit('handleDrop')
      },
      //列固定
      handleFixed(CurrentColumn,direction) {
        this.$emit('handleFixed',CurrentColumn,direction)
      },
      //取消固定
      cancelFixed(CurrentColumn) {
        this.$emit('cancelFixed',CurrentColumn)
      },
   }
 }
</script>

<style type="text/scss" lang="scss" scoped>
//列设置
 .tree-table-setting {
   flex: 1;
   display: flex;
   align-items: center;
   justify-content: space-between;
   font-size: 14px;
   padding-right: 8px;
 }
 //列头设置
 .custom-setting {
   padding: 5px 20px;
   margin-bottom: 10px;
   background-color: #f3f3f3;
   display: flex;
   justify-content: space-between;
   align-items: center;
 }
 .activatedFixed {
   color: #409eff;
 }
 .fixedActivated {
   color: #ccc;
 }
</style>