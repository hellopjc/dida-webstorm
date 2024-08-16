<template>
  <Modal v-model="edit" :title="pageTypeName + '修改'">
    <Form ref="form" :modal="form" :label-width="100" :rules="ruleCustom">
      <FormItem label="上级">{{ superiors }}</FormItem>
      <FormItem :label="pageTypeName+'名称'" prop="struName">
        <Input v-model="form.struName" type="text" placeholder="同目录下名称需要唯一"/>
      </FormItem>
      <FormItem label="URL" prop="struUrl">
        {{form.fullUri}}
      </FormItem>
      <FormItem label="类型">{{pageTypeName}}</FormItem>
      <FormItem label="显示顺序">
        <Input v-model="form.orders" type="number" style="width:300px"/>
      </FormItem>
    </Form>
    <!--    <Button @click="test()">测试</Button>-->
    <div slot="footer">
      <Button ghost type="primary" icon="ios-undo" @click="close">取消</Button>
      <Button type="primary" icon="ios-archive" @click="pageEdit" :loading='isloading'
              v-project="{projectId:this.projectId,permissionTypeList:[Owner,Maintainer]}">保存</Button>
    </div>
  </Modal>
</template>

<script>
import ajax from '@lambo-design/shared/utils/ajax'
import config from '@/config/config'
import {Owner,Maintainer,Reader} from "@/constant/project-permission";

export default {
  name: 'dir-menu-page-edit',
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
      edit: false,
      recievedData:{},
      struId: '',
      form: {
        projectId:'',
        struId: '',
        struName: '',
        struUrl: '',
        pageType: '01',
        isLeaf: '0',
        parentId: '',
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
              if (self.clickNodeInfoFromTree.level !== 4 && self.clickNodeInfoFromTree.children.length > 0){
                for (let item of self.clickNodeInfoFromTree.children) {
                  if (item.label == self.form.struName){
                    callback(new Error("节点名称已存在"))
                  }
                }
              } else if (self.clickNodeInfoFromTree.level === 4){
                const url = config.didaManageServerContext + '/manage/page/stru/list?parentId=' + this.clickNodeInfoFromTree.struId
                const response = this.syncRequest(url)
                const data = response.data.data
                console.log('why????',data)
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
        ]
      }
    };
  },
  watch: {
    struId() {
      this.getRest();
    }
  },
  computed: {
    superiors:function(){
      switch(this.clickNodeInfoFromTree.level) {
        case 2: return "应用";
        case 3: return "目录";
        case 4: return "菜单";
        default: return "";
      }
    },
    pageTypeName:function () {
      switch(this.clickNodeInfoFromTree.level) {
        case 2: return "目录";
        case 3: return "菜单";
        case 4: return "页面";
        default: return "";
      }
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
    getRest(arg) {
      console.log("arg:",arg)
      ajax.get(config.didaManageServerContext + '/manage/page/stru/get/' + arg.struId).then((resp) => {
        if (resp.data.code === 1) {
          this.form = {
            ...resp.data.data,
            struProperty: JSON.parse(resp.data.data.struProperty),
            fullUri: JSON.parse(resp.data.data.struProperty).fullUri
          };
          console.log("this.form:",this.form)
        }
      });
    },
    pageEdit() {
      var self = this;
      self.isloading = true;
      this.$refs.form.validate((valid) => {
        if (valid) {
          var struParams = {
            struName: self.form.struName,
            orders: self.form.orders,
          };
          //修改目录
          ajax.post(config.didaManageServerContext + "/manage/page/stru/update/" + self.form.struId, struParams).then(function (resp) {
            var result = resp.data;
            if (result.code === 1) {
              //修改服务
              if (self.clickNodeInfoFromTree.level === 3 || self.clickNodeInfoFromTree.level === 4) {
                var restParams = {
                  pageName: self.form.struName,
                  projectId: self.form.projectId
                }
                ajax.post(config.didaManageServerContext + "/manage/page/setting/update/" + self.form.pageId, restParams).then(function (resp) {
                  var result = resp.data;
                  self.isloading = false;
                  self.$Message.success('修改成功');
                  self.$emit('submitSuccess');
                  //更新树
                  self.$emit("updateStruTree");
                  self.close();
                  // if (result.code == '1') {
                  //   self.isloading = false;
                  //   self.$Message.success('修改成功');
                  //   self.$emit('submitSuccess');
                  //   //更新树
                  //   self.$emit("updateStruTree");
                  //   self.close();
                  // } else {
                  //   self.isloading = false;
                  //   self.$Message.error('修改失败');
                  // }
                });
              } else {
                self.isloading = false;
                self.$Message.success('修改成功');
                self.$emit('submitSuccess');
                self.$emit("updateStruTree");
                self.close();
              }
            } else if (result.code === 0 && result.data === -1) {
              self.isloading = false;
              self.$Message.error('当前资源已被锁定，请解锁后再操作。');
            } else {
              self.isloading = false;
              self.$Message.error('修改失败');
            }
          });
        }else{
          self.isloading = false;
        }
      })
    },
    open(rowData) {
      this.recievedData = rowData;
      this.edit = true;
      this.isloading = false;
      this.getRest(rowData);
    },
    close() {
      this.edit = false;
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
    }

  }
}
</script>
