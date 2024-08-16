<template>
  <div>
    <Modal v-model="showModal" :title="templateTitle" :scrollable="true" :footer-hide="true" :width="1240" :styles="{top:'10px'}">
      <div v-if="showModal" :style="{height: windowHeight + 'px'}"> <!--,overflow: 'auto'-->
        <div class="base-info">
          <LamboIndicatorCard>
            <div slot="content-title">基本信息</div>
            <Form ref="form" :modal="form" :label-width="100" :rules="ruleCustom">
              <FormItem label="上级" style="margin-bottom:2px;">{{ clickNodeInfoFromTree.label }}</FormItem>
              <FormItem :label="pageTypeName+'名称'" prop="struName">
                <Input v-model="form.struName" type="text" placeholder="同目录下名称需要唯一" style="width:450px"/>
              </FormItem>
              <FormItem label="URL" prop="struUrl">
                <Input v-model="form.struUrl" type="text" style="width:450px" placeholder="格式：/***，以‘/’开头字母数字任意组合 ">
                  <span slot="prepend">{{ clickNodeInfoFromTree.path }} </span>
                </Input>
              </FormItem>
              <FormItem label="类型" style="margin-bottom:2px;">{{pageTypeName}}</FormItem>
              <FormItem label="显示顺序" style="margin-bottom:5px;">
                <Input v-model="form.orders" type="number" style="width:450px"/>
              </FormItem>
            </Form>
          </LamboIndicatorCard>
        </div>

        <div style="margin-top: 10px;margin-bottom: 8px;" :style="{height: halfWindowHeight + 'px',overflow: 'auto'}">
          <LamboIndicatorCard>
            <div slot="content-title">选择模版</div>
            <div slot="content-extend">
              <Input v-model="searchKey" icon="ios-search" placeholder="请输入模版名称" style="width: 200px;margin-right: 10px;"/>
              <Button type="primary" @click="searchTemplate" style="margin-right: 10px;">搜索</Button>
              <Button type="default" @click="searchReset">重置</Button>
            </div>
            <Layout style="background: #FFFFFF">
              <Sider :width="240" style="height:100%">
                <Menu theme="light" active-name="1" @on-select="selectMenu">
                  <MenuItem name="0" class="font-bold">
                    全部模版
                  </MenuItem>
                  <MenuGroup title="系统模板">
                    <MenuItem name="1" class="font-bold">
                      <Icon type="md-desktop" />PC端
                    </MenuItem>
                    <MenuItem name="2" class="font-bold">
                      <Icon type="md-phone-portrait" />移动端
                    </MenuItem>
                  </MenuGroup>
                  <MenuGroup title="自定义模版">
                    <MenuItem name="3" class="font-bold">
                      <Icon type="md-desktop" />PC端
                    </MenuItem>
                    <MenuItem name="4" class="font-bold">
                      <Icon type="md-phone-portrait" />移动端
                    </MenuItem>
                  </MenuGroup>
                </Menu>
              </Sider>
              <Content :style="{padding: '24px', background: '#fff'}"><!--minHeight: '280px',-->
                <div class="page-temp-card-list">
                  <div class="inner">
                    <Card v-if="selectMenuTab==='0'" v-for="card in allTempList" :key="card.templateId" style="display:inline-block;margin:10px 10px 20px 10px;">
                      <div class="card-content" @click="cardSelected(card)">
                        <Icon class="check-icon" type="ios-checkbox" v-show="activeCard.templateId == card.templateId" />
                        <div class="content-wrapper">
                          <div :ref="'imgBox'+card.templateId" :class="'img-box ' + (hoverCard.templateId === card.templateId ? 'hover' : '')"
                               @mouseover="mouseEnterCard(card)" @mouseout="hoverCard = {}" @click="openImgModal(card)">
                            <img v-if="card.templateType==='1'" class="img" :src="card.picture" v-show="!!card.picture"/>
                            <LamboUploadImg v-else-if="card.templateType==='0'"
                                            v-show="!!card.picture"
                                            :oss-server-context="ossServerContext"
                                            :has-delete="false"
                                            :do-cut="false"
                                            :show-only="true"
                                            :default-img="card.picture"
                                            :preview-modal-width="60"
                                            :preview-img-height="500"
                                            :preview-img-width="800"
                            ></LamboUploadImg>
                            <span class="empty_tip" v-show="!card.picture">暂无图片</span>
                          </div>
                          <div class="temp_title" :title="card.templateName || '未命名'">{{card.templateName || '未命名'}}</div>
                        </div>
                      </div>
                    </Card>
                  </div>
                </div>
                <div class="page-temp-card-list">
                  <div class="inner">
                    <Card v-if="selectMenuTab==='1'" v-for="card in systemTempPCList" :key="card.templateId" style="display:inline-block;margin:10px 10px 20px 10px;">
                      <div class="card-content" @click="cardSelected(card)">
                        <Icon class="check-icon" type="ios-checkbox" v-show="activeCard.templateId == card.templateId" />
                        <div class="content-wrapper">
                          <div :ref="'imgBox'+card.templateId" :class="'img-box ' + (hoverCard.templateId === card.templateId ? 'hover' : '')"
                               @mouseover="mouseEnterCard(card)" @mouseout="hoverCard = {}" @click="openImgModal(card)">
                            <img class="img" :src="card.picture" v-show="!!card.picture"/>
                            <span class="empty_tip" v-show="!card.picture">暂无图片</span>
                          </div>
                          <div class="temp_title" :title="card.templateName || '未命名'">{{card.templateName || '未命名'}}</div>
                        </div>
                      </div>
                    </Card>
                  </div>
                </div>
                <div class="page-temp-card-list">
                  <div class="inner">
                    <Card v-if="selectMenuTab==='2'" v-for="card in systemTempMobileList" :key="card.templateId" style="display:inline-block;margin:10px 10px 20px 10px;">
                      <div class="card-content" @click="cardSelected(card)">
                        <Icon class="check-icon" type="ios-checkbox" v-show="activeCard.templateId == card.templateId" />
                        <div class="content-wrapper">
                          <div :ref="'imgBox'+card.templateId" :class="'img-box ' + (hoverCard.templateId === card.templateId ? 'hover' : '')"
                               @mouseover="mouseEnterCard(card)" @mouseout="hoverCard = {}" @click="openImgModal(card)">
                            <img class="img" :src="card.picture" v-show="!!card.picture"/>
                            <span class="empty_tip" v-show="!card.picture">暂无图片</span>
                          </div>
                          <div class="temp_title" :title="card.templateName || '未命名'">{{card.templateName || '未命名'}}</div>
                        </div>
                      </div>
                    </Card>
                  </div>
                </div>
                <div class="page-temp-card-list">
                  <div class="inner">
                    <Card v-if="selectMenuTab==='3'" v-for="card in customTempPCList" :key="card.templateId" style="display:inline-block;margin:10px 10px 20px 10px;">
                      <div class="card-content" @click="cardSelected(card)">
                        <Icon class="check-icon" type="ios-checkbox" v-show="activeCard.templateId == card.templateId" />
                        <div class="content-wrapper">
                          <div :ref="'imgBox'+card.templateId" :class="'img-box ' + (hoverCard.templateId === card.templateId ? 'hover' : '')"
                               @mouseover="mouseEnterCard(card)" @mouseout="hoverCard = {}" @click="openImgModal(card)">
                            <LamboUploadImg v-show="!!card.picture"
                                            :oss-server-context="ossServerContext"
                                            :has-delete="false"
                                            :do-cut="false"
                                            :show-only="true"
                                            :default-img="card.picture"
                                            :preview-modal-width="60"
                                            :preview-img-height="500"
                                            :preview-img-width="800"
                            ></LamboUploadImg>
                            <span class="empty_tip" v-show="!card.picture">暂无图片</span>
                          </div>
                          <div class="temp_title" :title="card.templateName || '未命名'">{{card.templateName || '未命名'}}</div>
                        </div>
                      </div>
                    </Card>
                  </div>
                </div>
                <div class="page-temp-card-list">
                  <div class="inner">
                    <Card v-if="selectMenuTab==='4'" v-for="card in customTempMobileList" :key="card.templateId" style="display:inline-block;margin:10px 10px 20px 10px;">
                      <div class="card-content" @click="cardSelected(card)">
                        <Icon class="check-icon" type="ios-checkbox" v-show="activeCard.templateId == card.templateId" />
                        <div class="content-wrapper">
                          <div :ref="'imgBox'+card.templateId" :class="'img-box ' + (hoverCard.templateId === card.templateId ? 'hover' : '')"
                               @mouseover="mouseEnterCard(card)" @mouseout="hoverCard = {}" @click="openImgModal(card)">
                            <LamboUploadImg v-show="!!card.picture"
                                            :oss-server-context="ossServerContext"
                                            :has-delete="false"
                                            :do-cut="false"
                                            :show-only="true"
                                            :default-img="card.picture"
                                            :preview-modal-width="60"
                                            :preview-img-height="500"
                                            :preview-img-width="800"
                            ></LamboUploadImg>
                            <span class="empty_tip" v-show="!card.picture">暂无图片</span>
                          </div>
                          <div class="temp_title" :title="card.templateName || '未命名'">{{card.templateName || '未命名'}}</div>
                        </div>
                      </div>
                    </Card>
                  </div>
                </div>
              </Content>
            </Layout>
          </LamboIndicatorCard>
        </div>
        <footer>
          <div class="ivu-modal-footer">
            <Button type="default" @click="modalCancel" style="margin-right:10px;"><span>取消</span></Button>
            <Button type="primary" @click="modalOk"><span>确定</span></Button>
          </div>
        </footer>
      </div>
    </Modal>

    <!-- 图片查看 -->
    <Modal v-model="imgModal" title="图片预览" width="60" :closable="false" style="text-align: center;" :styles="{top:'20px'}">
      <div style="width: 100%;">
        <img :src="previewImg" style="width: 90%;height: 480px;"/>
      </div>
      <div slot="footer">
        <Button type="primary" @click="closeImgModal">关闭</Button>
      </div>
    </Modal>
  </div>
</template>

<script>
import config from '@/config/config'
import { LamboUploadImg } from '@lambo-design/upload-img'
import LamboIndicatorCard from "@lambo-design/indicator-card";
import ajax from '@lambo-design/shared/utils/ajax'

export default {
  name: 'menu-page-create',
  components: {
    LamboUploadImg,
    LamboIndicatorCard
  },
  props: {
    value: {
      type: Boolean,
      default: false
    },
    allTemplateList:{
      type: Array,
      default: () => []
    },
    systemTemplatePCList: {
      type:Array,
      default: () => []
    },
    systemTemplateMobileList: {
      type:Array,
      default: () => []
    },
    customTemplatePCList: {
      type:Array,
      default: () => []
    },
    customTemplateMobileList: {
      type:Array,
      default: () => []
    },
    templateTitle: {
      type: String,
      default: ''
    },
    clickNodeInfoFromTree: {
      type: Object,
      required: true
    },
    appId: {
      type: String,
      default: ""
    },
    projectId: {
      type: String,
      default: ""
    },
  },
  data(){
    return {
      showModal: false,
      imgModal: false,
      activeCard: {},
      hoverCard: {},
      previewImg: "",
      ossServerContext: config.didaManageServerContext,
      create: false,
      form: {
        projectId:'',
        struId: '',
        struName: '',
        struUrl: '',
        pageType: '',
        isLeaf: '0',
        parentId: '',
        pageId: '',
        struPath: '',
        orders: 0,
        tenantId: ''
      },
      searchKey: '',
      selectMenuTab: '1',
      allTempList: [],
      customTempPCList: [],
      customTempMobileList: [],
      systemTempPCList: [],
      systemTempMobileList: [],
      ruleCustom: {
        struName: [
          {
            required: true,
            validator: (rule, value, callback) => {
              let self = this
              if (self.form.struName == '') {
                callback(new Error("节点名称不能为空"))
              }
              if (self.clickNodeInfoFromTree.level === 3  && self.clickNodeInfoFromTree.children && self.clickNodeInfoFromTree.children.length > 0) {
                for (let item of self.clickNodeInfoFromTree.children) {
                  if (item.label == self.form.struName) {
                    callback(new Error("节点名称已存在"))
                  }
                }
              } else if (self.clickNodeInfoFromTree.level === 4){
                const url = config.didaManageServerContext + '/manage/page/stru/list?parentId=' + this.clickNodeInfoFromTree.struId
                const response = this.syncRequest(url)
                const data = response.data.data
                // const struNameList = []
                for (let item of data.rows) {
                  if (item.struName == self.form.struName) {
                    //console.log('oh????')
                    callback(new Error("节点名称已存在"))
                  }
                }
              }
              callback()
            },
            trigger: 'blur'
          }
        ],
        struUrl: [
          {
            required: true,
            validator: (rule, value, callback) => {
              let self = this
              if (self.form.struUrl == '') {
                callback(new Error("节点URL不能为空"))
              }
              if (!self.form.struUrl.startsWith('/')) {
                callback(new Error("节点URL必须以‘/’开头"))
              }
              var reg = new RegExp(/^\w+$/);
              if (!reg.test((self.form.struUrl).substr(1))) {
                callback(new Error("节点URL格式不正确"))
              }
              if (self.clickNodeInfoFromTree.level === 3  && self.clickNodeInfoFromTree.children && self.clickNodeInfoFromTree.children.length > 0){
                for (let item of self.clickNodeInfoFromTree.children) {
                  if (item.path === (self.clickNodeInfoFromTree.path + self.form.struUrl)){
                    callback(new Error("节点URL已存在"))
                  }
                }
              } else if (self.clickNodeInfoFromTree.level === 4){
                  const url = config.didaManageServerContext + '/manage/page/stru/list?parentId=' + this.clickNodeInfoFromTree.struId
                  const response = this.syncRequest(url)
                  const data = response.data.data
                  // const struNameList = []
                  for (let item of data.rows) {
                    if (item.struUrl == self.form.struUrl) {
                      //console.log('oh????')
                      callback(new Error("节点URL已存在"))
                    }
                  }
              }
              callback()
            },
            trigger: 'blur'
          }
        ]
      }
    }
  },
  computed: {
    pageTypeName() {
      if (this.clickNodeInfoFromTree) {
        if (this.clickNodeInfoFromTree.level === 2) {
          // this.form.pageType = '01';
          return "目录";
        }
        if (this.clickNodeInfoFromTree.level === 3) {
          // this.form.pageType = '02';
          return "菜单";
        }
        if (this.clickNodeInfoFromTree.level === 4) {
          // this.form.pageType = '03';
          return "页面";
        }
      }
      // this.form.pageType = '01';
      return "";
    },
    windowHeight() {
      return (window.innerHeight * 0.88);
    },
    halfWindowHeight(){
      return (window.innerHeight * 0.47);
    }
  },
  methods: {
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
    // open(){
    //   this.showModal = true;
    // },
    modalOk(){
      // if (Object.keys(this.activeCard).length) {
      //   this.$emit('on-ok', {...this.activeCard});
      // }
      var self = this;
      this.$refs.form.validate((valid) => {
        if (valid) {
          console.log("valid")
          if (self.form.pageType == '02' || self.form.pageType == '03') {
            self.form.isLeaf = '1';
          }
          var struParams = {
            projectId: self.projectId,
            struName: self.form.struName,
            struUrl: self.form.struUrl,
            pageType: self.form.pageType,
            isLeaf: self.form.isLeaf,
            appId: self.clickNodeInfoFromTree.appId,
            parentId: self.clickNodeInfoFromTree.struId,
            orders: self.form.orders,
            struProperty: JSON.stringify({
              fullUri: self.clickNodeInfoFromTree.path + self.form.struUrl
            }),
            pageUri: self.clickNodeInfoFromTree.path + self.form.struUrl,
            pageDesign: ''
          }
          console.log('*********',struParams)
          let pageTemplate = {...this.activeCard}
          if (pageTemplate.templateId){
            struParams.pageDesign = pageTemplate.templateDesign
          }
          //console.log("struParams",struParams)
          // 新增目录，
          ajax.post(config.didaManageServerContext + "/manage/page/stru/create", struParams).then(function (resp) {
            const result = resp.data;
            console.log('111',result.code);
            if (result.code === 1) {
              //更新树
              //console.log('111');
              if (self.clickNodeInfoFromTree.level === 3){
                self.$emit('updateStruTree')
              }
              self.$Message.success('新增成功');
              self.$emit('submitSuccess');
            } else {
              //console.log('222');
              self.$Message.error('新增失败');
            }
          }).catch(e => {
            console.error(e)
            self.$Message.error('新增失败');
          });
        }
      })
      this.modalCancel();
    },
    open(pageId) {
      this.form.pageId = pageId
      this.create = true;
    },
    close() {
      this.create = false;
      this.searchKey = ''
      this.form = {
        projectId: '',
        struId: '',
        struName: '',
        struUrl: '',
        pageType: '',
        isLeaf: '0',
        parentId: '',
        pageId: '',
        struPath: '',
        orders: 0,
        tenantId: ''
      };
    },
    modalCancel() {
      this.showModal = false
      this.close()
    },
    cardSelected(card) {
      if (card.templateId !== this.activeCard.templateId) {
        this.activeCard = card;
      }
    },
    openImgModal(card){
      // 有图片才可查看
      if (card.picture) {
        this.previewImg = card.picture;
        this.imgModal = true;
      }
    },
    closeImgModal(){
      this.imgModal = false
      this.previewImg = ''
    },
    mouseEnterCard(card){
      if (card.picture) {
        this.hoverCard = card;
      }
    },
    searchTemplate(){
      let params = {
        limit: 1000,
        templateName: this.searchKey
      }
      ajax.get(config.didaManageServerContext + '/manage/pageTemplate/list',{params}).then(res => {
        if (res.data.code === 1){
          let result = res.data.data.rows
          this.allTempList = result;
          this.customTempPCList = result.filter(item => (item.templateType === '0' && item.templateTag === '0'))
          this.customTempMobileList = result.filter(item => (item.templateType === '0' && item.templateTag === '1'))
          this.systemTempPCList = result.filter(item => (item.templateType === '1' && item.templateTag === '0'))
          this.systemTempMobileList = result.filter(item => (item.templateType === '1' && item.templateTag === '1'))
        }
      })
    },
    searchReset(){
      this.searchKey = ''
      this.customTempPCList = this.customTemplatePCList
      this.systemTempPCList = this.systemTemplatePCList
      this.customTempMobileList = this.customTemplateMobileList
      this.systemTempMobileList = this.systemTemplateMobileList
      this.allTempList = this.allTemplateList
    },
    selectMenu(a){
      this.selectMenuTab = a
    }
  },
  watch: {
    value(val, oldVal) {
      this.showModal = val;
    },
    showModal(val, oldVal){
      this.$emit('input', val);
    },
    systemTemplatePCList(val, oldVal) {
      this.systemTempPCList = val;
    },
    systemTemplateMobileList(val, oldVal) {
      this.systemTempMobileList = val;
    },
    customTemplatePCList(val, oldVal) {
      this.customTempPCList = val;
    },
    customTemplateMobileList(val, oldVal) {
      this.customTempMobileList = val;
    },
    allTemplateList(val, oldVal) {
      this.allTempList = val
    }
  }
}
</script>

<style lang="less" scoped>
.font-bold{
  font-weight: bold;
}
.page-temp-card-list {
  .inner {
    display: flex;
    flex-wrap: wrap;
    .ivu-card {
      position: relative;
      margin: 15px;
      cursor: pointer;
      &.active {
        border-color: #2d8cf0
      }
      /deep/ .ivu-card-body {
        padding: 0;
      }
      .check-icon {
        position: absolute;
        right: 0;
        top: 0;
        margin-right: 0;
        color: #2d8cf0;
        font-size: 20px;
      }
    }
  }
  .content-wrapper {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: center;
    width: 150px;
    padding: 18px 18px 12px 18px;
    box-sizing: unset;
    user-select: none;

    .img-box {
      position: relative;
      display: flex;
      justify-content: center;
      align-items: center;
      margin-bottom: 6px;
      width: 100%;
      height: 150px;
      overflow: hidden;
      .img {
        width: 100%;
        height: 100%;
      }
      .empty_tip {
        color: #ddd;
      }
      &::before {
        content: '查看';
        position: absolute;
        left: 0;
        top: -100%;
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100%;
        width: 100%;
        transition: top .3s ease;
        background: #0003;
        color: #eee;
      }
      &.hover {
        &::before {
          top: 0;
        }
      }
    }

    .temp_title {
      width: 100%;
      font-size: 18px;
      //text-align: center;
      white-space: nowrap;
      text-overflow: ellipsis;
      overflow: hidden;
    }
  }
}
</style>
