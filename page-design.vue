<template>
  <LamboPageContainer :has-footer="false">
    <template slot="page-title">
      {{ title }}
    </template>
    <template slot="page-extend">
      <div style="display: flex;align-items: center;">
        <span>运行期账号: {{runtimeAccount}}</span>
        <Button type="primary" ghost size="small" @click="runtimeAccountChange" style="margin-left: 10px;font-size: 14px;">切换账号</Button>
      </div>
    </template>
    <Row>
      <Col span="4" style="padding-left: 5px">
        <LeftTree :tree-data="treeData"
                  v-on:getProjectId="getProjectIdFromTree"
                  v-on:getClickNodeInfo="getClickNodeInfoFromTree"
                  :project-id="projectId">
        </LeftTree>
      </Col>
      <Col span="20" style="padding-left: 15px">
        <RightAllCards :click-node-info-from-tree="clickNodeInfoFromTree"
                       :default-tree-data="treeData"
                       :init-info="initInfo" @updateStruTree="updateStruTree">

        </RightAllCards>
      </Col>
    </Row>
  </LamboPageContainer>
</template>

<script>
import LamboPageContainer from "@lambo-design/page-container";
import LeftTree from '@/view/design-center/left-tree.vue'
import ajax from '@lambo-design/shared/utils/ajax'
import config from '@/config/config'
import Json from '@lambo-design/shared/utils/json'
import { nanoid } from 'nanoid'
import RightAllCards from '@/view/design-center/right-all-cards.vue'

export default {
  name:'page-design',
  components:{
    RightAllCards,
    LamboPageContainer,
    LeftTree
  },
  data(){
    return{
      projectId: '',
      projectName: '',  //需要从left-tree传过来
      projectList: [],
      treeData: [],
      clickNodeInfoFromTree: {},
      initInfo: {
        projectId: '',
        projectName: '',
        projectDesc: ''
      }
    }
  },
  created(){
    const projectIdFromUrl = this.$route.query.id;
    if (projectIdFromUrl) {
      this.projectId = projectIdFromUrl;
    }
    //获取传递给left-tree的projectList
    this.getProjectList()

    this.getTreeData()
  },
  beforeDestroy() {

  },
  mounted() {
    //将projectList通过全局事件总线传给left-tree
    this.$bus.$emit('getProjectList',this.projectList)
    // this.$bus.$emit('getTreeData',this.treeData)

    //
    //this.$bus.$on('getProjectId',(data) => {this.projectId=data})
    //console.log('fu*****',this.projectId)
  },

  computed: {
    title: function () {
      return this.$route.meta.title
    }
  },
  watch: {
    projectId(){
      this.getTreeData()

      //传给right-all-card初始化信息
      //this.initInfo.projectName
    }
  },

  methods: {
    //right-all-cards中变化时触发
    updateStruTree(){
      console.log('updateStruTreeFu')
      this.getTreeData()
    },
    getClickNodeInfoFromTree(data){
      this.clickNodeInfoFromTree = data
      //console.log('fuClickNode',typeof data,data)
    },

    getProjectIdFromTree(data){
      this.projectId = data
      //console.log('fu',this.projectId)
      this.$router.push({
        name: '页面设计',
        query: {
          id: this.projectId
        }
      })
      //this.getTreeData()
    },
    //获取传递给left-tree的projectList
    getProjectList(){
      const self = this
      //在此程序中，需先获得projectList再全局事件总线，所以不要使用异步的ajax发请求！！！
      const response = this.syncRequest(config.didaManageServerContext + "/manage/project/listNoPage")
      console.log('projectList--response:',response);
      const projectList = response.data.data;
      if (projectList != null && projectList.length > 0) {
        self.projectList = projectList;
        console.log('getProjectList:',self.projectList);
      }
    },
    //同步请求，与getProjectList配合使用
    syncRequest(url){
      var xhr = new XMLHttpRequest();
      xhr.open('GET', url, false); //false为同步请求
      xhr.send(null)
      var response = {
        status: xhr.status,
        statusText: xhr.statusText,
        data: null
      }
      if (xhr.status === 200) {
        response.data = JSON.parse(xhr.responseText)
      } else {
        throw new Error('请求失败，状态码：' + xhr.status);
      }
      return response;
    },

    //获取left-tree中需要的treeData
    getTreeData(){
      let treeData = []
      let node = {}
      //获取倒数第三级节点（2级节点）
      const resp = this.syncRequest(config.didaManageServerContext
        + '/manage/application/list?projectId='
        + this.projectId + '&limit=' + 100)
      if (resp.data && resp.data.code === 1) {
        const data = resp.data.data
        console.log('111111',data)
        for (let item of this.projectList){
          if (item.projectId === this.projectId) {
            this.projectName = item.projectName
            this.initInfo.projectDesc = item.projectDesc
            console.log('****9898*****',this.initInfo.projectDesc)
            break
          }
        }
        // ajax.get(config.didaManageServerContext + '/manage/project/list?projectId=' + this.projectId).then(res => {
        //   this.initInfo.projectDesc = res.data.data.projectDesc
        //   console.log('99986997',res.data.data,this.initInfo.projectDesc)
        // })
        //传给right-all-card初始化信息
        this.initInfo.projectName = this.projectName
        this.initInfo.projectId = this.projectId
        node.path = '/'
        node.label = this.projectName
        node.level = 1
        node.id = 1
        if (data && data.rows && data.rows.length > 0) {
          //进此if，则表明有2级节点，如默认项目中的test2
          node.children = []
          //此处已经得到2级节点数据，如默认项目中的test2，获取倒数第二级节点（3级节点）
          for (let item of data.rows){
            let node2level = {}
            //console.log('*****item',item)
            node2level.path = item.path
            node2level.level = 2
            node2level.label = item.appName
            node2level.id = nanoid(5)
            node2level.appId = item.appId
            //同步请求每一个二级分支，获取其子节点，如默认项目中的test2的子节点
            const url = config.didaManageServerContext
              + "/manage/page/stru/queryChildren?parentId=" + 'root' + "&projectId=" + this.projectId
              + '&appId=' + item.appId

            ajax.get(url).then(response => {
              if (response.data && response.data.data.length > 0) {
                //进此if，则表明有3级节点，如默认项目的test2的增删改查
                node2level.children = []
                //获取最后一级节点（4级节点）
                const dataSon = response.data.data
                for (let itemSon of dataSon){
                  let node3level = {}
                  //console.log('%%%%%%%%%%',itemSon)
                  node3level.path = node2level.path + itemSon.struUrl
                  node3level.level = 3
                  node3level.label = itemSon.struName
                  node3level.id = nanoid(5)
                  node3level.struId = itemSon.struId
                  node3level.appId = item.appId
                  //console.log('node3level.label:',node3level.label)
                  const urlSon = config.didaManageServerContext
                    + "/manage/page/stru/queryChildren?parentId=" + itemSon.struId + "&projectId=" + this.projectId
                    + '&appId=' + item.appId

                  ajax.get(urlSon).then(responseSon => {
                    if (responseSon.data && responseSon.data.data.length > 0) {
                      //进此if，则表明有4级节点，如默认项目的test2的增删改查的用户管理
                      node3level.children = []
                      const dataLast = responseSon.data.data
                      for (let itemLast of dataLast){
                        let node4level = {}
                        node4level.level = 4
                        node4level.path = node3level.path + itemLast.struUrl
                        //console.log('******itemLast',itemLast)
                        node4level.label = itemLast.struName
                        node4level.id = nanoid(5)
                        node4level.struId = itemLast.struId
                        node4level.appId = item.appId
                        node3level.children.push(node4level)
                      }
                      ////////
                      node2level.children.push(node3level)
                    }else {
                      node2level.children.push(node3level)
                    }
                  })
                  //console.log('************************respSon',responseSon)
                }
                //////
                node.children.push(node2level)
              } else {
                node.children.push(node2level)
              }
            })
            //console.log(response)
          }
          treeData.push(node)
        }else {
          treeData.push(node)
        }
        console.log('8888888888888',treeData)
        //this.treeData = treeData
        this.$set(this.treeData, 0, treeData[0])
      } else {
        this.$Message.error('获取页面数量失败')
      }
    },
  },
}
</script>

<style scoped lang="less">
/deep/.table-option-dropdown {
  //display: none !important;
  visibility:hidden;
}
/deep/.ivu-tree {
  overflow: auto; /* 自动滚动条 */
  height: calc(100vh - 0px);
}
.banner {
  background-image: url('../../assets/images/page-design/page-design-banner.png');
  background-size: cover; /* 覆盖整个容器 */
  background-position: center; /* 居中背景图片 */
}

/deep/.content-body {
  background: none !important; /* 或者使用 transparent */
}

/* 修改 ViewUI Select 组件的下拉滚动条样式 */
.ivu-select-dropdown .ivu-select-dropdown-list {
  /* 这是整个下拉列表的滚动容器 */
  scrollbar-width: thin; /* 对于 Firefox */
  scrollbar-color: grey transparent; /* 对于 Firefox, 设置滚动条和轨道颜色 */

  /* 对于 Chrome, Edge 和 Safari */
  ::-webkit-scrollbar {
    width: 5px; /* 滚动条的宽度 */
    height: 5px; /* 滚动条的高度，对横向滚动条有效 */
  }
  ::-webkit-scrollbar-track {
    background: transparent; /* 滚动条轨道的颜色 */
  }
  ::-webkit-scrollbar-thumb {
    background-color: grey; /* 滚动条的颜色 */
    border-radius: 10px; /* 滚动条的圆角 */
  }
}

</style>
