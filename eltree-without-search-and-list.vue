<template>
  <LamboIndicatorCard :has-title="false" :has-extend="false">
    <el-input class="el-input" placeholder="请输入关键字过滤" v-model="filterText"></el-input>
    <div class="left">
      <el-tree
        :data="thisTreeData"
        node-key="id"
        :default-expanded-keys="[1]"
        @node-click="handleNodeClick"
        :filter-node-method="filterNode"
        ref="tree"
        style="height:100%; margin-top:30px; background-color:#FFF;"
      >
       <span class="custom-tree-node" slot-scope="{ node, data }">
        <span v-if="data.level === 1" class="projectIcon"> </span>
        <span v-else-if="data.level === 2" class="appIcon" ></span>
        <span v-else-if="data.level === 3" class="dirIcon" ></span>
        <span v-else-if="data.level === 4" class="menuIcon" ></span>
        <span v-if="data.level !==4" class="el-tree-node__label1">{{node.label}}</span>
        <span v-else class="el-tree-node__label2">{{node.label}}</span>
        <span class="letterStyle">({{data.path}})</span>
      </span>
      </el-tree>
    </div>
  </LamboIndicatorCard>

</template>

<script>
import LamboIndicatorCard from '@lambo-design/indicator-card'

export default {
  name: 'eltree-without-search-and-list',
  props: {
    treeData: {
      type: Array,
      required: true
    },
    projectId: {
      type: String,
      required: true
    },
    treeDefaultExpandedKey: {
      type: Array,
      required: false
    }
  },
  components: {
    LamboIndicatorCard,
  },
  data() {
    return {
      projectId: '',
      projectList: [],
      filterText: '',
      defaultProps: {
        children: 'children',
        label: 'label'
      },
      thisTreeData: [],
    }
  },


  created() {
    //this.$bus.$on('getProjectList', this.getProjectListFromBus);

    ////
    // this.$bus.$on('getTreeData', this.getTreeData);
    //console.log('*************treedata', this.treeData);
    this.thisTreeData = this.treeData;
  },
  mounted() {

  },
  watch: {
    filterText(val) {
      this.$refs.tree.filter(val);
    },
    // thisTreeData:{
    //   immediate: true,
    //   deep: true,
    // }
  },
  beforeDestroy() {

  },
  methods: {
    //左树过滤判断
    filterNode(value, data) {
      if (!value) return true;
      //console.log(value,data,(data.label.indexOf(value) !== -1 || data.path.indexOf(value) !== -1))
      return (data.label.indexOf(value) !== -1 || data.path.indexOf(value) !== -1);
    },
    handleNodeClick(node) {
      this.$emit('getClickNodeInfo',node)
      console.log(node)
    },

    getTreeData(data){
      this.treeData = data;
      console.log('999999999999999999999',this.treeData);
    },

    handleProjectChange (val) {
      const self = this
      self.projectId = val
      //this.$bus.$emit('getProjectId', self.projectId)
      this.$emit('getProjectId',self.projectId)
      console.log('zi',self.projectId)
    }
  }


}
</script>

<style scoped>
.el-input{
  margin-top: 20px;
  margin-bottom: -10px;
  font-size: 12px;
  width: 100%
}
  .left {
    height: calc(100vh + 10px);
    //width: 100%;
    //overflow: scroll;
  }
  :deep(.el-tree){
    /* border: 1px solid #dcdfe6; */
    width: 100%;
    //display: inline-block;
    overflow: scroll;
    //margin-top: 12px;

  }
  :deep(.el-tree>.el-tree-node) {
    display: inline-block;
    min-width: 100%;
  }
  .el-tree-node__label1 {
    width: auto;
    margin-right: 0;
    font-size: 12px;
    color: #333333 ;
  }
  .el-tree-node__label2 {
    margin-left: 0 !important;
    margin-right: 0 !important;
    font-size: 12px;
    color: #ff0000 ;
  }

  .projectIcon{
    background: url("项目.png")
    center no-repeat;
    background-size: 12px 12px;
    //margin-left: 10px;
    margin-right: 0;
    padding: 12px 12px;
  }
  .appIcon{
    background: url("应用.png")
    center no-repeat;
    background-size: 12px 12px;
    //margin-left: 10px;
    margin-right: 0;
    padding: 12px 12px;
  }
  .dirIcon{
    background: url("目录.png")
    center no-repeat;
    background-size: 12px 12px;
    //margin-left: 10px;
    margin-right: 0;
    padding: 12px 12px;
  }
  .menuIcon{
    background: url("菜单.png")
    center no-repeat;
    background-size: 12px 12px;
    //margin-left: 10px;
    margin-right: 0;
    padding: 12px 12px;
  }
  .letterStyle{
    //margin-left: 10px;
    font-family: "KaiTi", Sans-serif;
    color: grey;
    font-size: 10px;
  }
</style>
