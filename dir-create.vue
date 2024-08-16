<template>
  <Modal v-model="create" :title="'新增'+pageTypeName">
    <Form ref="form" :modal="form" :label-width="100" :rules="ruleCustom">
      <FormItem label="上级">{{ clickNodeInfoFromTree.label }}</FormItem>
      <FormItem :label="pageTypeName+'名称'" prop="struName">
        <Input v-model="form.struName" type="text" placeholder="同目录下名称需要唯一"/>
      </FormItem>
      <FormItem label="URL" prop="struUrl">
        <Input v-model="form.struUrl" type="text" style="width:300px" placeholder="格式：/***，以‘/’开头字母数字任意组合 ">
          <span slot="prepend">{{ clickNodeInfoFromTree.path }} </span>
        </Input>
      </FormItem>
      <FormItem label="类型">{{pageTypeName}}</FormItem>
      <FormItem label="显示顺序">
        <Input v-model="form.orders" type="number" style="width:300px"/>
      </FormItem>
    </Form>
    <!--    <Button @click="test()">测试</Button>-->
    <div slot="footer">
      <Button ghost type="primary" icon="ios-undo" @click="close">取消</Button>
      <Button type="primary" icon="ios-archive" @click="pageSubmit" :loading='isloading'  v-project="{projectId:this.projectId,permissionTypeList:[Owner,Maintainer]}">保存</Button>
    </div>
  </Modal>
</template>

<script>
import ajax from '@lambo-design/shared/utils/ajax'
import config from '@/config/config'
import {Owner,Maintainer,Reader} from "@/constant/project-permission";

export default {
  name: 'dir-create',
  props:{
    clickNodeInfoFromTree:{
      type: Object,
      required: true
    },
    projectId: {
      type: String,
      default: "",
      required: true
    },

    pageTemplate: {
      type: Object,
      default: {}
    }
  },
  data() {
    let self = this
    return{
      Owner,
      Maintainer,
      Reader,
      isloading: false,
      create: false,
      pageTypeName: '目录',
      form: {
        projectId:'',
        struId: '',
        struName: '',
        struUrl: '',
        pageType: '01',
        isLeaf: '0',
        parentId: 'root',
        pageId: '',
        struPath: '',
        orders: 0,
        tenantId: ''
      },
      ruleCustom: {
        struName: [
          {
            required: true,
            validator: (rule, value, callback) => {
              if (self.form.struName == '') {
                callback(new Error("节点名称不能为空"))
              }
              if (self.clickNodeInfoFromTree.children && self.clickNodeInfoFromTree.children.length > 0){
                for (let item of self.clickNodeInfoFromTree.children) {
                  if (item.label == self.form.struName){
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
              if (self.clickNodeInfoFromTree.children && self.clickNodeInfoFromTree.children.length > 0){
                for (let item of self.clickNodeInfoFromTree.children) {
                  if (item.path === (self.clickNodeInfoFromTree.path + self.form.struUrl)){
                    callback(new Error("节点名称已存在"))
                  }
                }
              }
              callback()
            },
            trigger: 'blur'
          }
        ]
      }
    };
  },
  mounted() {
    this.init()
  },
  methods: {
    init(){
      this.getTenantId()
    },
    pageSubmit() {
      const self = this
      self.isloading = true;
      console.log("valid?")
      this.$refs.form.validate((valid) => {
        if (valid) {
          console.log("valid")
          let struParams = {
            projectId: self.projectId,
            struName: self.form.struName,
            struUrl: self.form.struUrl,
            pageType: self.form.pageType,
            isLeaf: self.form.isLeaf,
            appId: self.clickNodeInfoFromTree.appId,
            parentId: self.form.parentId,
            orders: self.form.orders,
            struProperty: JSON.stringify({
              fullUri: self.clickNodeInfoFromTree.path + self.form.struUrl
            }),
            pageUri: self.clickNodeInfoFromTree.path + self.form.struUrl,
          };
          console.log("struParams",struParams)
          // 新增目录，
          ajax.post(config.didaManageServerContext + "/manage/page/stru/create", struParams).then(function (resp) {
            var result = resp.data;
            if (result.code == '1') {
              //更新树
              self.$emit('update-stru-tree')
              self.isloading = false;
              self.$Message.success('新增成功');
              self.$emit('submit-success');
              self.close();
            } else {
              self.isloading = false;
              self.$Message.error('新增失败');
            }
          }).catch(e => {
            console.error(e)
            self.$Message.error('新增失败');
            self.isloading = false;
          });
        } else {
          self.isloading = false;
        }
      })

    },
    getTenantId(){
      ajax.get(config.didaManageServerContext + "/manage/project/get/" + this.projectId).then((resp) => {
        if (resp.data.code === 1) {
          this.form.tenantId = resp.data.data.tenantId
        } else {
          this.$Message.error("数据获取异常，请稍候再试")
        }
      })
    },
    open(pageId) {
      this.form.pageId = pageId
      this.create = true;
      this.isloading = false;
    },
    close() {
      this.create = false;
      this.form = {
        projectId: '',
        struId: '',
        struName: '',
        struUrl: '',
        pageType: '01',
        isLeaf: '0',
        parentId: 'root',
        pageId: '',
        struPath: '',
        orders: 0,
        tenantId: ''
      };
    }

  }
}
</script>
