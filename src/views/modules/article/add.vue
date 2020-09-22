<template>
  <div class="articleAdd-wrapper">
    <a-card :body-style="{padding: '24px 32px'}" :bordered="false">
      <a-form :form="detailsForm">
        <a-form-item
          label="标题"
          :labelCol="{lg: {span: 4}, sm: {span: 4}}"
          :wrapperCol="{lg: {span: 6}, sm: {span: 17} }"
        >
          <a-input
            v-decorator="[
              'title',
              validatorRules.title
            ]"
            name="title"
            placeholder="请输入标题"
          />
        </a-form-item>
        <a-form-item
          label="关键字"
          :labelCol="{lg: {span: 4}, sm: {span: 4}}"
          :wrapperCol="{lg: {span: 6}, sm: {span: 17}}"
        >
          <a-input
            v-decorator="[
              'keyword'
            ]"
            name="keyword"
            placeholder="请输入关键字"
          />
        </a-form-item>
        <a-form-item
          label="摘要"
          :labelCol="{lg: {span: 4}, sm: {span: 4}}"
          :wrapperCol="{lg: {span: 6}, sm: {span: 17} }"
        >
          <a-input
            v-decorator="[
              'digest',
              validatorRules.digest
            ]"
            name="digest"
            placeholder="请输入关键字"
          />
        </a-form-item>
        <a-form-item
          label="文章类型"
          :labelCol="{lg: {span: 4}, sm: {span: 4}}"
          :wrapperCol="{lg: {span: 6}, sm: {span: 17} }"
        >
          <a-select placeholder="请选择广告类型" v-decorator="[ 'typeCode', validatorRules.typeCode]">
            <a-select-option v-for="(item, index) in type" :value="item.itemValue">{{item.itemText}}</a-select-option>
          </a-select>
        </a-form-item>
        <a-form-item
          label="文章封面"
          :labelCol="{lg: {span: 4}, sm: {span: 4}}"
          :wrapperCol="{lg: {span: 6}, sm: {span: 17} }"
        >
          <a-upload
            listType="picture-card"
            class="avatar-uploader"
            :showUploadList="false"
            :beforeUpload="frontBeforeUpload"
            @change="frontHandleChange"
          >
            <img v-if="details.img" :src="details.img" style="height:100px;max-width:300px" />
            <div v-else>
              <a-icon :type="uploadLoading ? 'loading' : 'plus'" />
              <div class="ant-upload-text">上传</div>
            </div>
          </a-upload>
        </a-form-item>
        <a-form-item
          label="内容"
          :labelCol="{lg: {span: 4}, sm: {span: 4}}"
          :wrapperCol="{lg: {span: 18}, sm: {span: 24} }"
        >
          <vue-ueditor-wrap v-model="details.content" :config="myConfig" :destroy="true"></vue-ueditor-wrap>
        </a-form-item>
        <a-form-item
          label="是否推荐"
          :labelCol="{lg: {span: 4}, sm: {span: 4}}"
          :wrapperCol="{lg: {span: 18}, sm: {span: 24} }"
        >
          <a-switch v-decorator="['switch', { valuePropName: 'checked', initialValue: true }]" />
        </a-form-item>
      </a-form>
      <div class="add-btn">
        <a-button type="primary" @click="submit">提交</a-button>
        <!-- <a-button>取消</a-button> -->
      </div>
    </a-card>
  </div>
</template>

<script>
import VueUeditorWrap from 'vue-ueditor-wrap'
import { getAction } from '@/api/manage'
import pick from 'lodash.pick'
export default {
  data() {
    return {
      detailsForm: this.$form.createForm(this),
      uploadLoading: false,
      type: [],
      details: {
        img: '',
        content: ''
      },
      validatorRules: {
        typeCode: { rules: [{ required: true, message: '请选择文章类型' }] },
        title: { rules: [{ required: true, message: '请输入标题' }] },
        digest: { rules: [{ required: true, message: '请输入摘要' }] }
      },
      myConfig: {
        // 编辑器不自动被内容撑高
        autoHeightEnabled: false,
        // 初始容器高度
        initialFrameHeight: 240,
        // 初始容器宽度
        initialFrameWidth: '100%',
        serverUrl: process.env.VUE_APP_BASE_API + '/system/ueditor/action'
      },
      articleId: '',
      formMsg: {}
    }
  },
  components: {
    VueUeditorWrap
  },
  created() {
    this.articleId = this.$route.query.id
    if (this.articleId) {
      this.searchDetails()
    }
    this.searchType()
  },
  methods: {
    // 查询文章详情
    searchDetails() {
      const url = '/system/sysArticle/queryById'
      const params = {}
      params.id = this.articleId
      return getAction(url, params).then(res => {
        if (res.success) {
          let result = res.result
          console.log(result)
          this.formMsg = result
          this.details.img = result.img
          this.details.content = result.content
          this.$nextTick(() => {
            this.detailsForm.setFieldsValue(pick(this.formMsg, 'typeCode', 'title', 'keyword', 'digest'))
            this.detailsForm.setFieldsValue({
              switch: result.isRecommend
            })
          })
        }
      })
    },
    // 获取文章类型
    searchType() {
      const url = '/sys/dictItem/list'
      const params = {}
      params.dictId = 'e897cd65d67a18c0695c45abbea768fe'
      params.field = 'id,,itemText,itemValue,action'
      params.pageNo = '1'
      params.pageSize = '50'
      return getAction(url, params).then(res => {
        if (res.success) {
          let result = res.result
          // console.log(result)
          this.type = result.records
        }
      })
    },
    // 上传 bofore
    frontBeforeUpload: function(file) {
      return false
    },
    // 上传 change
    frontHandleChange(info) {
      // console.log(info)
      this.upload(info, 'img')
    },
    // upload
    upload(info, name) {
      let that = this
      let reader = new FileReader()
      reader.readAsDataURL(info.file)
      reader.onload = function() {
        that.base64Img = reader.result
        that.base64Img = encodeURI(that.base64Img)
        // encodeURI(URIstring)
        that.axios
          .post('/common/uploadbase64img', {
            folder: 'admin/',
            base64Img: that.base64Img
          })
          .then(res => {
            // console.log(res)
            that.details[name] = res.result
          })
          .catch(err => {
            console.log(err)
          })
      }
    },
    add(values) {
      this.axios({
        method: 'post',
        url: '/system/sysArticle/add',
        data: {
          ...values,
          isRecommend: values.switch,
          img: this.details.img,
          content: this.details.content
        }
      })
        .then(res => {
          console.log(res)
          if (res.success) {
            this.$router.push({
              path: '/article/list'
            })
          }
        })
        .catch(err => {
          console.log(err)
          this.$message.error('新增失败，请重试')
        })
    },
    edit(values) {
      this.axios({
        method: 'post',
        url: '/system/sysArticle/edit',
        data: {
          ...values,
          id: this.articleId,
          isRecommend: values.switch,
          img: this.details.img,
          content: this.details.content
        }
      })
        .then(res => {
          console.log(res)
          if (res.success) {
            this.$router.push({
              path: '/article/list'
            })
          } else {
            this.$message.error(res.message)
          }
        })
        .catch(err => {
          console.log(err)
          this.$message.error('编辑失败，请重试')
        })
    },
    // 提交
    submit() {
      this.detailsForm.validateFields((err, values) => {
        if (!err) {
          // eslint-disable-next-line no-console
          console.log('Received values of form: ', values)
          if (this.articleId) {
            this.edit(values)
          } else {
            this.add(values)
          }
        } else {
          console.log('other')
        }
      })
    }
  }
}
</script>

<style lang="scss">
.articleAdd-wrapper {
  position: relative;
  .add-btn {
    margin-left: 16.6%;
    margin-top: 30px;
    button + button {
      margin-left: 15px;
    }
  }
}
.edui-default  .edui-for-importword  .edui-icon {
  background-position: -380px 0px;
}
@import '../../../assets/less/135editor.less';
@import '../../../assets/less/importword.less';
</style>