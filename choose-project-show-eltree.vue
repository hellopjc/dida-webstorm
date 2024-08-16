<template>
  <Modal
    v-model="showPageCopyModal"
    title="请选择一个菜单"
    :width="600"
    :scrollable=true
    :footer-hide=true
  >
    <Select v-model="localProjectId" class="el-input">
      <Option v-for="item in projectList" :value="item.projectId" :key="item.projectId">{{ item.projectName }}</Option>
    </Select>

    <EltreeWithoutSearchAndList :tree-data="treeData"
                                @getClickNodeInfo="getClickNodeInfoFromTree"
                                :project-id="localProjectId">
    </EltreeWithoutSearchAndList>

    <footer>
      <div class="ivu-modal-footer">
        <button type="button" class="ivu-btn ivu-btn-text ivu-btn-large" @click="onCancel"><span>取消</span></button>
        <button type="button" class="ivu-btn ivu-btn-primary ivu-btn-large" @click="onOk"  v-project="{projectId:this.projectId,permissionTypeList:[Owner,Maintainer]}"><span>确定</span></button>
      </div>
    </footer>
  </Modal>
</template>

<script>
import config from "@/config/config";
import ajax from "@lambo-design/shared/utils/ajax";
import {Owner,Maintainer,Reader} from "@/constant/project-permission";
import EltreeWithoutSearchAndList from '@/view/design-center/eltree-without-search-and-list.vue'
import { nanoid } from 'nanoid'

export default {
  name: "choose-project-show-eltree",
  components: { EltreeWithoutSearchAndList },
  props: {
    value: {
      type: Boolean,
      default: false
    },
    projectId:{
      type: String,
      default: ''
    },
    projectList: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      showPageCopyModal: this.value,
      appId: '',
      localProjectId: '',
      copyStru: {},
      appList: [],
      treeData: [],
      clickNodeInfoFromTree: {},
      Owner,
      Maintainer,
      Reader,
    }
  },
  created() {
    this.localProjectId = this.projectId
    this.getTreeData()
  },
  updated() {

  },
  watch: {
    localProjectId(){
      this.getTreeData()
    },

    value: {
      handler: function (val) {
        this.showPageCopyModal = val;
      }
    },
    showPageCopyModal: {
      handler: function (val) {
        this.$emit("input",val);
      }
    }
  },
  methods: {
    getClickNodeInfoFromTree(data){
      this.clickNodeInfoFromTree = data
      //console.log('fuClickNode',typeof data,data)
    },
    getTreeData(){
      let treeData = []
      let node = {}
      //获取倒数第三级节点（2级节点）
      const resp = this.syncRequest(config.didaManageServerContext
        + '/manage/application/list?projectId='
        + this.localProjectId + '&limit=' + 100)
      if (resp.data && resp.data.code === 1) {
        const data = resp.data.data
        console.log('111111',data)
        for (let item of this.projectList){
          if (item.projectId === this.localProjectId) {
            node.label = item.projectName
            break
          }
        }
        console.log('222222')
        node.level = 1
        node.id = 1
        node.path = '/'
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

            //同步请求每一个二级分支，获取其子节点，如默认项目中的test2的子节点
            const url = config.didaManageServerContext
              + "/manage/page/stru/queryChildren?parentId=" + 'root' + "&projectId=" + this.localProjectId
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

                  //console.log('node3level.label:',node3level.label)
                  const urlSon = config.didaManageServerContext
                    + "/manage/page/stru/queryChildren?parentId=" + itemSon.struId + "&projectId=" + this.localProjectId
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

    onOk() {
      if (this.clickNodeInfoFromTree.level !== 4) {
        this.$Message.error("请选择一个菜单")
      } else {
        this.showPageCopyModal = false;
        this.$emit("on-ok", this.clickNodeInfoFromTree);
      }
    },
    onCancel() {
      this.showPageCopyModal = false;
    },
  },

}
</script>

<style lang="less" scoped>
.ivu-tree {
  height: 400px;
  overflow: auto;
}
.el-input{
  width: 100% ;
  margin-top: 0;
  margin-bottom: -10px;
  font-size: 12px;
}
</style>
