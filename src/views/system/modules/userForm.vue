<template>
  <a-modal
    :title="() => {return model && model.id > 0 ? '编辑用户信息' : '新增用户' }"
    :width="640"
    :visible="visible"
    :confirmLoading="loading"
    ok-text="提交"
    cancel-text="取消"
    @ok="() => { $emit('ok') }"
    @cancel="() => { $emit('cancel') }"
  >
    <a-spin :spinning="loading">
      <a-form :form="form" v-bind="formLayout">
        <a-input v-show="false" v-decorator="['id', { initialValue: model && model.id }]" disabled/>
        <a-form-item label="账号">
          <a-input
            v-decorator="['name', {rules: [{initialValue: model && model.name, required: true, min: 3, max: 255, message: '请输入至少三个字符的账号！'}]}]"/>
        </a-form-item>
        <a-form-item v-show="!model" label="密码">
          <a-input-password v-decorator="['password', {required: true, min: 8, max: 255, message: '请输入至少八个字符的密码！' }]"/>
        </a-form-item>
        <a-form-item label="用户名称">
          <a-input
            v-decorator="['userName', {rules: [{initialValue: model && model.userName, required: true, max: 255, message: '最多输入255个字符的用户名称！'}]}]"/>
        </a-form-item>
        <a-form-item label="手机号码">
          <a-input
            v-decorator="['phone', {rules: [{initialValue: model && model.phone, required: true, message: '请输入手机号码！'},{validator:phoneCheck.bind(this)}]}]"/>
        </a-form-item>
        <a-form-item label="E-mail">
          <a-input
            v-decorator="['email',{rules: [{type: 'email',message: '请输入正确email!'}], initialValue: model && model.email}]"
          />
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import pick from 'lodash.pick'
  import { checkPhone } from '@/api/manage'

  // 表单字段
  const fields = ['description', 'id']

  export default {
    props: {
      visible: {
        type: Boolean,
        required: true
      },
      loading: {
        type: Boolean,
        default: () => false
      },
      model: {
        type: Object,
        default: () => null
      }
    },
    data () {
      this.formLayout = {
        labelCol: {
          xs: { span: 24 },
          sm: { span: 7 }
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 13 }
        }
      }
      return {
        form: this.$form.createForm(this)
      }
    },
    created () {
      console.log('custom modal created')

      // 防止表单未注册
      fields.forEach(v => this.form.getFieldDecorator(v))

      // 当 model 发生改变时，为表单设置值
      this.$watch('model', () => {
        this.model && this.form.setFieldsValue(pick(this.model, fields))
      })
    },
    methods: {
      phoneCheck (rule, value, callbackFn) {
        if (!value) {
          callbackFn()
          return
        }

        const reg = /^1\d{10}$/
        if (value && !reg.test(value)) {
          callbackFn('请输入正确手机号码！')
          return
        }

        checkPhone(value).then(res => {
          if (res.result !== 0) {
            callbackFn('手机号码已经绑定其他账号！')
          } else {
            callbackFn()
          }
        })
      }
    }
  }
</script>
