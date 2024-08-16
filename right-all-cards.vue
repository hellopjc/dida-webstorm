<template>
  <div>
    <LamboIndicatorCard :has-title="false" :has-extend="false" class="banner">
      <div v-if="clickNodeInfoFromTree.id && clickNodeInfoFromTree.id !== 1">
        <h1>{{clickNodeInfoFromTree.label}}</h1>
        <p>目标链接：{{clickNodeInfoFromTree.path}}</p>
      </div>
      <div v-if="!clickNodeInfoFromTree.id || clickNodeInfoFromTree.id === 1">
        <h1>{{initInfo.projectName}}</h1>
        <p>项目描述：{{initInfo.projectDesc}}</p>
      </div>
    </LamboIndicatorCard>
    <LamboIndicatorCard :has-title="false" :has-extend="false">

      <LamboQueryFilter v-if="!clickNodeInfoFromTree.level || clickNodeInfoFromTree.level === 1" @do-search="doAppSearch" @on-reset="onAppRest">
        <FormItem label="应用名称">
          <Input v-model="appSearchForm.appName" placeholder="请输入应用名称"/>
        </FormItem>
      </LamboQueryFilter>

      <LamboQueryFilter v-if="clickNodeInfoFromTree.level === 2" @do-search="doSearch" @on-reset="onRest">
        <FormItem label="目录名称">
          <Input v-model="searchForm.struName" placeholder="请输入目录名称"/>
        </FormItem>
        <FormItem label="状态" style="margin-left: 5px;">
          <Select v-model="searchForm.status" placeholder="请选择状态" clearable>
            <Option v-for="item in stateList" :value="item.value" :key="item.value">{{ item.label }}</Option>
          </Select>
        </FormItem>
      </LamboQueryFilter>

      <LamboQueryFilter v-if="clickNodeInfoFromTree.level === 3" @do-search="doSearch" @on-reset="onRest">
        <FormItem label="菜单名称">
          <Input v-model="searchForm.struName" placeholder="请输入菜单名称"/>
        </FormItem>
        <FormItem label="状态" style="margin-left: 5px;">
          <Select v-model="searchForm.status" placeholder="请选择状态" clearable>
            <Option v-for="item in stateList" :value="item.value" :key="item.value">{{ item.label }}</Option>
          </Select>
        </FormItem>
      </LamboQueryFilter>

      <LamboQueryFilter v-if="clickNodeInfoFromTree.level === 4" @do-search="doSearch" @on-reset="onRest">
        <FormItem label="页面名称">
          <Input v-model="searchForm.struName" placeholder="请输入页面名称"/>
        </FormItem>
        <FormItem label="状态" style="margin-left: 5px;">
          <Select v-model="searchForm.status" placeholder="请选择状态" clearable>
            <Option v-for="item in stateList" :value="item.value" :key="item.value">{{ item.label }}</Option>
          </Select>
        </FormItem>
      </LamboQueryFilter>
    </LamboIndicatorCard>

    <LamboIndicatorCard :has-title="false" :has-extend="false">
<!--      应用list-->
      <LamboVxePagingTable
        v-if="!clickNodeInfoFromTree.level || clickNodeInfoFromTree.level === 1"
        :data-url="dataUrl"
        :search-params="appSearchParams"
        ref="xTable"
      >
        <div slot="buttons">
          <Button type="primary" icon="md-add"
                  @click="appCreateModal" style="margin-right: 5px" ghost
                  v-permission="'design-center:page-directory-group:create'"
                  v-project="{projectId:this.initInfo.projectId,permissionTypeList:[Owner,Maintainer]}">
            新增应用
          </Button>
        </div>
        <vxe-column field="appName" title="应用名称" align="left" show-overflow min-width="150" fixed="left"></vxe-column>
        <vxe-column field="appTitle" title="应用标题" align="left" show-overflow min-width="150"></vxe-column>
        <vxe-column field="path" title="应用路径" align="left" show-overflow min-width="150"></vxe-column>
        <vxe-column field="status" title="应用状态" align="left" :formatter="formatTypeApp" show-overflow min-width="150"></vxe-column>
        <vxe-column field="action" title="操作" align="left" show-overflow min-width="150" fixed="right">
          <template #default="{ row }">
            <a href="#" v-if="clickNodeInfoFromTree.level === 1 && row.isLock !== '1'" @click.prevent="appCreateModal(row)" style="margin-right: 5px"
               v-permission="'design-center:page-directory-group:edit'" v-project="{projectId:row.projectId,permissionTypeList:[Owner,Maintainer,Reader]}">编辑</a>
            <a href="#" v-if="clickNodeInfoFromTree.level !== 1 && row.isLock !== '1'" @click.prevent="editModal(row)" style="margin-right: 5px"
               v-permission="'design-center:page-directory-group:edit'" v-project="{projectId:row.projectId,permissionTypeList:[Owner,Maintainer,Reader]}">编辑</a>
            <a href="#" v-if="clickNodeInfoFromTree.level === 3 || clickNodeInfoFromTree.level === 4" @click.prevent="checkRuntimeAccount(row)" style="margin-right: 5px"
               v-permission="'design-center:page:design'" v-project="{projectId:row.projectId,permissionTypeList:[Owner,Maintainer,Reader]}">页面设计</a>
            <a href="#" v-if="clickNodeInfoFromTree.level === 1" @click.prevent="handleAppDelete(row.appId)" style="margin-right: 5px"
               v-permission="'design-center:page-directory-group:delete'" v-project="{projectId:row.projectId,permissionTypeList:[Owner,Maintainer]}">删除</a>
            <a href="#" v-if="row.isLock !== '1' && clickNodeInfoFromTree.level !== 1" @click.prevent="doDelete(row)" style="margin-right: 5px"
               v-permission="'design-center:page-directory:delete'||'design-center:page:delete'" v-project="{projectId:row.projectId,permissionTypeList:[Owner,Maintainer]}">删除</a>

            <a href="#" v-if="row.isLock === '0' || row.isLock === null" @click.prevent="pageLock(row)" style="margin-right: 5px"
               v-permission="'design-center:page-directory:lock'||'design-center:page:lock'" v-project="{projectId:row.projectId,permissionTypeList:[Owner,Maintainer]}">锁定</a>
            <a href="#" v-if="row.isLock === '1'" @click.prevent="pageUnLock(row)" style="margin-right: 5px"
               v-permission="'design-center:page-directory:unlock'||'design-center:page:unlock'" v-project="{projectId:row.projectId,permissionTypeList:[Owner,Maintainer]}">解锁</a>

            <a href="#" v-if="clickNodeInfoFromTree.level === 1" @click="doCheck(row.appId)"
               v-permission="'design-center:page-directory-group:sql'" v-project="{projectId:row.projectId,permissionTypeList:[Owner,Maintainer,Reader]}">导出</a>
            <a href="#" v-else @click="doCheck(row.struId)"
               v-permission="'design-center:page-directory:sql'||'design-center:page:sql'" v-project="{projectId:row.projectId,permissionTypeList:[Owner,Maintainer,Reader]}">导出</a>
          </template>
        </vxe-column>
      </LamboVxePagingTable>

<!--      目录、菜单、页面list-->
      <LamboVxePagingTable
        v-if="clickNodeInfoFromTree.level === 2 || clickNodeInfoFromTree.level === 3 || clickNodeInfoFromTree.level === 4"
        :data-url="dataUrl"
        :search-params="searchParams"
        ref="xTable"
      >
        <div slot="buttons">
          <Button type="primary" v-if="clickNodeInfoFromTree.level === 2" icon="md-add"
                  @click="createModal" style="margin-right: 5px" ghost
                  v-permission="'design-center:page-directory-group:create'"
                  v-project="{projectId:this.initInfo.projectId,permissionTypeList:[Owner,Maintainer]}">
            新增目录
          </Button>
          <Button type="primary" v-if="clickNodeInfoFromTree.level === 3" icon="md-add"
                  @click="createMenuOrPage" style="margin-right: 5px" ghost
                  v-permission="'design-center:page-directory-group:create'"
                  v-project="{projectId:this.initInfo.projectId,permissionTypeList:[Owner,Maintainer]}">
            新增菜单
          </Button>
          <Button type="primary" v-if="clickNodeInfoFromTree.level === 4" icon="md-add"
                  @click="createMenuOrPage" style="margin-right: 5px" ghost
                  v-permission="'design-center:page-directory-group:create'"
                  v-project="{projectId:this.initInfo.projectId,permissionTypeList:[Owner,Maintainer]}">
            新增页面
          </Button>
          <Button type="primary" v-if="clickNodeInfoFromTree.level === 3" icon="md-add"
                  @click="openCopyModal" style="margin-right: 5px" ghost
                  v-permission="'design-center:page-directory-group:create'"
                  v-project="{projectId:this.initInfo.projectId,permissionTypeList:[Owner,Maintainer]}">
            复制目录
          </Button>
        </div>
        <vxe-column v-if="clickNodeInfoFromTree.level === 2" field="struName" title="目录名称" align="left" show-overflow min-width="150" fixed="left"></vxe-column>
        <vxe-column v-if="clickNodeInfoFromTree.level === 2" field="struUrl" title="目录链接" align="left" show-overflow min-width="150"></vxe-column>
        <vxe-column v-if="clickNodeInfoFromTree.level === 3" field="struName" title="菜单名称" align="left" show-overflow min-width="150" fixed="left"></vxe-column>
        <vxe-column v-if="clickNodeInfoFromTree.level === 3" field="struUrl" title="菜单链接" align="left" show-overflow min-width="150"></vxe-column>
        <vxe-column v-if="clickNodeInfoFromTree.level === 4" field="struName" title="页面名称" align="left" show-overflow min-width="150" fixed="left"></vxe-column>
        <vxe-column v-if="clickNodeInfoFromTree.level === 4" field="struUrl" title="页面链接" align="left" show-overflow min-width="150"></vxe-column>

        <vxe-column field="state" title="状态" align="left" :formatter="formatTypePage" show-overflow min-width="40"></vxe-column>
        <vxe-column field="createTime" title="创建时间"  :formatter="dateFormat" align="left" show-overflow min-width="120"></vxe-column>
        <vxe-column field="action" title="操作" align="left" show-overflow min-width="200" fixed="right"></vxe-column>
      </LamboVxePagingTable>
    </LamboIndicatorCard>
    <AppCreate
      ref="appCreate"
      :project-id="this.initInfo.projectId"
      @updateStruTree="updateStruTree"
      @submitSuccess="doAppSearch"
    ></AppCreate>
    <DirCreate ref="dirCreate" :click-node-info-from-tree="this.clickNodeInfoFromTree"
               :project-id="this.initInfo.projectId"
               @update-stru-tree="updateStruTree"
               @submit-success="doSearch"
    ></DirCreate>
    <DirMenuPageEdit
      ref="dirMenuPageEdit"
      :project-id="this.initInfo.projectId"
      :click-node-info-from-tree="this.clickNodeInfoFromTree"
      @updateStruTree="updateStruTree"
      @submitSuccess="doSearch"
    ></DirMenuPageEdit>

    <MenuPageCreate
      ref="pageTemplateModal"
      v-model="showPageTemplateModal"
      :click-node-info-from-tree="this.clickNodeInfoFromTree"
      :template-title="pageTemplateTitle"
      :all-template-list="allTemplateList"
      :system-template-p-c-list="systemTemplatePCList"
      :system-template-mobile-list="systemTemplateMobileList"
      :custom-template-p-c-list="customTemplatePCList"
      :custom-template-mobile-list="customTemplateMobileList"
      :project-id="this.initInfo.projectId"
      @updateStruTree="updateStruTree"
      @submitSuccess="doSearch"
    ></MenuPageCreate>
    <ChooseProjectShowEltree
      ref="copy"
      v-model="showPageCopyModal"
      :project-id="this.initInfo.projectId"
      :project-list="this.projectList"
      @on-ok="onOk"
    ></ChooseProjectShowEltree>
    <CopyModalFromChooseNode
      ref="copyModal"
      v-model="copyModal"
      :click-node-info-from-tree="this.clickNodeInfoFromTree"
      :project-id="this.initInfo.projectId"
      @updateStruTree="updateStruTree"
      @submitSuccess="doSearch"
    ></CopyModalFromChooseNode>
    <PageAccountModal :is-show="showRuntimeAccountModal"
                      @on-cancel="runtimeAccountModalCancel"
                      @login-but-no-cache="loginButNoCache"
                      @return-runtime-account-id="runtimeAccountUserId"></PageAccountModal>
  </div>

</template>


<script>
import LamboIndicatorCard from '@lambo-design/indicator-card'
import LamboQueryFilter from '@lambo-design/query-filter'
import LamboVxePagingTable from '@lambo-design/vxe-paging-table'
import AppCreate from '@/view/design-center/components/app-create.vue'
import ajax from '@lambo-design/shared/utils/ajax'
import config from '@/config/config'
import {Owner,Maintainer,Reader} from "@/constant/project-permission";
import { timestampToTime } from '@lambo-design/shared/utils/date'
import PageCreate from '@/view/design-center/components/page-create.vue'
import DirMenuPageEdit from '@/view/design-center/dir-menu-page-edit.vue'

import DirCreate from '@/view/design-center/dir-create.vue'
import { isRuntimeAccount } from '@/util/runtimeAccount'
import PageAccountModal from '@/view/design-center/components/page-account-modal.vue'
import MenuPageCreate from '@/view/design-center/menu-page-create.vue'
import Copy from '@/view/design-center/components/copy.vue'
import CopyModal from '@/view/design-center/components/copyModal.vue'
import ChooseProjectShowEltree from '@/view/design-center/choose-project-show-eltree.vue'
import CopyModalFromChooseNode from '@/view/copy-modal-from-choose-node.vue'
export default {
  name: 'right-all-cards',
  props: {
    defaultTreeData: {
      type: Array
    },
    clickNodeInfoFromTree: {
      type: Object,
      required: true
    },
    initInfo: {
      type: Object,
      required: true
    }
  },
  components: {
    CopyModal,
    Copy,
    PageAccountModal,
    PageCreate,
    AppCreate,
    LamboIndicatorCard,
    LamboQueryFilter,
    LamboVxePagingTable,
    DirCreate,
    DirMenuPageEdit,
    MenuPageCreate,
    ChooseProjectShowEltree,
    CopyModalFromChooseNode
  },
  data(){
    return{
      appSearchParams: {
        appName: '',
        //status: ''
      },
      appSearchForm: {
        appName: '',
        //status: ''
      },
      searchParams: {
        struName: '',
        status: ''
      },
      searchForm: {
        struName: '',
        status: ''
      },
      stateList: [
        {
          value: '10',
          label: '启用'
        },
        {
          value: '00',
          label: '禁用'
        }
      ],
      dataUrl: '',
      Owner,
      Maintainer,
      Reader,
      showRuntimeAccountModal: false,
      loginNoCache: false,
      showPageTemplateModal: false,
      allTemplateList: [],
      systemTemplatePCList: [],
      customTemplatePCList: [],
      customTemplateMobileList: [],
      systemTemplateMobileList: [],
      pageTemplateTitle: '',
      showPageCopyModal: false,
      copyModal: false,
      projectList: [],
    }
  },
  created() {
    this.$bus.$on('getProjectList', this.getProjectListFromBus);
  },
  watch: {
    clickNodeInfoFromTree: {
      handler(newVal){
        console.log('change')
        if (newVal.level === 1) {
          this.dataUrl = config.didaManageServerContext + '/manage/application/list?projectId=' + this.initInfo.projectId + '&sort=create_time'
        }else if(newVal.level === 2) {
          this.dataUrl = config.didaManageServerContext + "/manage/page/stru/list?parentId=" + 'root' + '&appId=' + this.clickNodeInfoFromTree.appId
          //console.log(this.dataUrl)
        }else if(newVal.level === 3 || newVal.level === 4){
          this.dataUrl = config.didaManageServerContext + '/manage/page/stru/list?parentId=' + this.clickNodeInfoFromTree.struId
          //console.log(this.dataUrl)
        }
      }
    },
    initInfo: {
      deep: true,
      handler(newVal){
        this.dataUrl = config.didaManageServerContext + '/manage/application/list?projectId=' + newVal.projectId + '&sort=create_time'
      }
    }
  },
  mounted() {
    this.dataUrl = config.didaManageServerContext + '/manage/application/list?projectId=' + this.initInfo.projectId
  },
  methods: {
    //使用全局事件总线从page-design获得projectList
    getProjectListFromBus(data){
      console.log('getProjectListFromBus',data);
      this.projectList = data;
    },

    onOk(node) {
      // this.showPage('copy')
      //this.$emit("on-copy-modal-ok", copyStruId);
      this.showPageCopyModal = false;
      this.$refs.copyModal.open(node);
    },
    openCopyModal() {
      this.showPageCopyModal = true;
    },
    /* 新增菜单或页面 */
    createMenuOrPage(){
      if(this.clickNodeInfoFromTree.level === 3){
        this.pageTemplateTitle = '新增菜单'
      }else if(this.clickNodeInfoFromTree.level === 4){
        this.pageTemplateTitle = '新增页面'
      }
      this.showPageTemplateModal = true;
      this.getCustomTemplateList()
    },
    getCustomTemplateList() {
      ajax.get(config.didaManageServerContext + '/manage/pageTemplate/list?limit=1000').then(res => {
        if (res.data.code === 1){
          let result = res.data.data.rows
          this.allTemplateList = result
          this.customTemplatePCList = result.filter(item => (item.templateType === '0' && item.templateTag === '0'))
          this.customTemplateMobileList = result.filter(item => (item.templateType === '0' && item.templateTag === '1'))
          this.systemTemplatePCList = result.filter(item => (item.templateType === '1' && item.templateTag === '0'))
          this.systemTemplateMobileList = result.filter(item => (item.templateType === '1' && item.templateTag === '1'))
        }
      })
    },

    doDelete(arg) {
      let self = this;
      let struId = arg.struId;
      let struName = arg.struName;
      let pageType = arg.pageType;
      let contentStr = "";
      if (self.clickNodeInfoFromTree.level === 4) {
        contentStr = "<span style='color:#ff0000;font-size:16px;padding:0 5px;'>确定要删除页面：" + struName + "吗？</span>";
      } else if (self.clickNodeInfoFromTree.level === 3) {
        contentStr = "<span style='color:#ff0000;font-size:16px;padding:0 5px;'>确定要删除菜单：" + struName + "吗？如果存在下级，请先删除所有下级！</span>";
      } else if (self.clickNodeInfoFromTree.level === 2){
        contentStr = "<span style='color:#ff0000;font-size:16px;padding:0 5px;'>确定要删除目录：" + struName + "吗？如果存在下级，请先删除所有下级！</span>";
      }
      self.$Modal.confirm({
        title: '页面删除',
        content: contentStr,
        onOk: () => {
          ajax.get(config.didaManageServerContext + "/manage/page/stru/delete/" + struId).then(function (resp) {
            const result = resp.data
            if (result.code === 1) {
              self.$Message.success('删除成功!');
              self.doSearch()
              self.updateStruTree();
            }else if (result.code === 2) {
              console.error(result.message)
              self.$Message.error('删除失败!请先删除所有下级');
            }else if (result.code === 3) {
              console.error(result.message)
              self.$Message.error('删除失败!请解锁后删除');
            } else {
              console.error(result.message)
              self.$Message.error('删除失败!请联系系统管理员');
            }
          }).catch(function (err) {
            console.error(err)
            self.$Message.error('删除失败,请联系系统管理员');
          });
        }
      });
    },

    checkRuntimeAccount(row) {
      setTimeout(() => {
        // 登录但没缓存，只对当前会话有效
        if (this.loginNoCache){
          this.goPage(row)
          return
        }
        if (!isRuntimeAccount()) {
          this.$Modal.confirm({
            title: '信息维护提示',
            content: '运行期账号未维护，页面组件无法获取动态数据！\n\n是否前往维护？',
            onOk: () => {
              this.showRuntimeAccountModal = true;
            }
          })
        }else {
          // 运行期账号已维护，跳转页面
          this.goPage(row)
        }
      }, 500)
    },
    goPage(row) {
      console.log('row')
      const href = window.location.origin + config.amisEditorContext + "/#/page/" + row.pageId
      window.open(href, '_blank');
    },
    runtimeAccountModalCancel(){
      this.showRuntimeAccountModal = false
    },
    loginButNoCache(param) {
      this.loginNoCache = param;
    },
    runtimeAccountUserId(param){
      this.runtimeAccount = param
    },

    createModal(pageId) {
      this.$refs.dirCreate.open(pageId);
    },

    updateStruTree(){
      console.log('updateStruTree')
      this.$emit('updateStruTree')
    },
    //状态转变
    formatTypeApp({ cellValue }) {
      const typeMap = {
        10: '启用',
        20: '删除',
      };
      return typeMap[cellValue] || '未知';
    },
    formatTypePage({ cellValue }){
      const typeMap = {
        10: '启用',
        '00': '禁用',
      };
      return typeMap[cellValue] || '未知';
    },
    dateFormat(e){
      return timestampToTime(e.cellValue)
    },
    /* 检查当前应用下页面是否锁定 */
    doCheck(struId) {
      const self = this
      ajax.get(config.didaManageServerContext + "/manage/page/export/isExistUnLock/" + struId).then(function (resp) {
        if (resp.data.code === 1 && resp.data.data === true) {
          self.$Modal.confirm({
            title: '提示',
            content: '<p>当前节点下存在未锁定节点，确定要锁定所有节点并导出吗?</p>',
            onOk: () => {
              ajax.post(config.didaManageServerContext + "/manage/page/export/updateIsLockStatus/" + struId, { "isLock": "1" }).then(function (resp) {
                if (resp.data.code === 1 && resp.data.data === true) {
                  self.exportSql(struId)
                } else {
                  self.$Message.error('锁定失败');
                }
              })
            }
          });
        } else {
          self.exportSql(struId)
        }
      });
    },
    /* 导出SQL */
    exportSql(struId){
      const url = config.didaManageServerContext + "/manage/page/export/getSql/" + struId
      ajax.get(url, {responseType: 'blob'}).then(function (res) {
        let blob = new Blob([res.data]);
        let url = window.URL.createObjectURL(blob); // 创建 url 并指向 blob
        let a = document.createElement('a');
        a.href = url;
        a.download = '数据服务.txt';
        a.click();
        window.URL.revokeObjectURL(url); // 释放该 url
      });
    },
    appCreateModal(rowData){
      this.$refs.appCreate.open(rowData)
    },
    editModal(rowData) {
      this.$refs.dirMenuPageEdit.open(rowData);
    },
    pageLock(arg) {
      const self = this
      //console.log('22222222222222')
      this.$Modal.confirm({
        title: '提示',
        content: '<p>确定要锁定吗?</p>',
        onOk: () => {
          ajax.post(config.didaManageServerContext + "/manage/page/setting/update/" + arg.struId, {"isLock": "1","projectId":this.initInfo.projectId}).then(function (resp) {
            console.log("arg.struId:" , arg.struId)
            console.log("arg:",arg)
            if (resp.data.code === 1) {
              self.$Message.success('锁定成功');
              self.doSearch()
              //self.getRest()
            } else {
              self.$Message.error('锁定失败');
            }
          });
        }
      });
    },
    pageUnLock(arg) {
      const self = this
      //console.log("self.curStru.isLock:",self.curStru.isLock)
      this.$Modal.confirm({
        title: '提示',
        content: '<p>确定要解锁吗?</p>',
        onOk: () => {
          ajax.post(config.didaManageServerContext + "/manage/page/setting/update/" + arg.struId, {"isLock": "0","projectId":this.initInfo.projectId}).then(function (resp) {
            if (resp.data.code === 1) {
              self.$Message.success('解锁成功');
              self.doSearch()
              //self.getRest()
            } else {
              self.$Message.success('解锁失败');
            }
          });
        }
      });
    },
    handleAppDelete(appId){
      var self = this;
      this.$Modal.confirm({
        title: '提示',
        content: '<p>确定要删除吗?</p>',
        onOk: () => {
          ajax.get(config.didaManageServerContext + "/manage/application/delete/" + appId).then(function (resp) {
            self.$Message.success('删除成功');
            self.updateStruTree();
            self.doAppSearch();
          })
        }
      });
    },
    doAppSearch() {
      this.appSearchParams = Object.assign({}, this.appSearchForm)
    },
    onAppRest(){
      this.appSearchForm.appName = ''
      //this.appSearchForm.status = ''
      this.doAppSearch()
    },
    doSearch(){
      this.searchParams = Object.assign({}, this.searchForm)
    },
    onRest(){
      this.searchForm.struName = ''
      this.searchForm.status = ''
      this.doSearch()
    }
  }
}
</script>


<style scoped>

</style>
