<template>
  <el-card class="box-card">
    <div slot="header" class="clearfix">
      <span>管理员变更</span>
    </div>
    <div class="box-body">
      <el-form
        :model="ruleForm"
        ref="ruleForm"
        :rules="rules"
        label-width="100px"
        label-position="right"
        :show-message="true"
        class="demo-ruleForm"
        v-if="!page.submitFormState.showState"
      >
        <el-form-item label="手机号" prop="phone" v-if="page.isChangeManage===3">
          <el-input type="text" :disabled="true" size="medium" v-model="ruleForm.phone"></el-input>
        </el-form-item>
        <el-form-item label="用户手机号:" v-if="page.isChangeManage===4">
          <el-autocomplete
            v-model="page.keyword"
            :fetch-suggestions="querySearchAsync"
            placeholder="请输入用户手机号"
            @select="handleSelect"
          >
            <template slot-scope="{ item }">
              <span class="userPhone">{{ item.phone }}</span>
              <span style="float: right;">{{ item.name }}</span>
            </template>
          </el-autocomplete>
        </el-form-item>
        <el-form-item label="用户名:" prop="name" v-if="hasNewManage">
          <label class="el-form-item__label">{{page.newManager.name}}</label>
        </el-form-item>
        <el-form-item
          label="图片验证码:"
          prop="imageCode"
          class="imageCode"
          v-if="page.isChangeManage===3"
        >
          <image-verify-code v-model="ruleForm.imageCode" key="imageCode1"/>
        </el-form-item>
        <el-form-item label="手机验证码:" prop="phoneCode" v-if="page.isChangeManage===3">
          <phone-verify-code
            v-model="ruleForm.phoneCode"
            key="phoneCode1"
            :is-change-manage="page.isChangeManage"
            :phone="ruleForm.phone"
            :img-code="ruleForm.imageCode"
          />
        </el-form-item>
        <el-form-item
          label="图片验证码:"
          prop="newimageCode"
          class="imageCode"
          v-if="page.isChangeManage===4"
        >
          <image-verify-code
            v-model="ruleForm.newimageCode"
            propName="newimageCode"
            key="imageCode2"
          />
        </el-form-item>
        <el-form-item label="手机验证码:" v-if="page.isChangeManage===4">
          <phone-verify-code
            v-model="ruleForm.newphoneCode"
            propName="newphoneCode"
            key="phoneCode2"
            :is-change-manage="page.isChangeManage"
            :phone="page.newManager.phone"
            :img-code="ruleForm.newimageCode"
          />
        </el-form-item>
        <el-form-item
          label="登录密码"
          prop="password"
          placeholder="请输入登录密码"
          v-if="page.isChangeManage===3"
        >
          <el-input type="password" size="medium" v-model="ruleForm.password"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="nextStep">下一步</el-button>
        </el-form-item>
      </el-form>
      <div class="tip" v-if="page.submitFormState.showState">
        <div class="showTip">
          <span class="showIcq">
            <i
              v-bind:class="[page.submitFormState.showIcon,page.submitFormState.isActive ? page.submitFormState.isSuccess:page.submitFormState.isError]"
            ></i>
          </span>
          <span
            style="font-size:25px;padding-left: 5px;vertical-align: text-bottom;"
            class="icon-name"
          >{{page.submitFormState.showText}}</span>
        </div>
        <div class="showBtn">
          <el-button type="primary" @click="goPage">{{page.submitFormState.btnText}}</el-button>
        </div>
      </div>
    </div>
  </el-card>
</template>

<script>
import { clearLoginInfo } from '@/utils'
import ImageVerifyCode from '../../shared/image-verify-code.vue'
import PhoneVerifyCode from '../../shared/phone-verify-code.vue'

export default {
  components: {
    ImageVerifyCode,
    PhoneVerifyCode
  },
  data () {
    const validateImageCode = (rule, value, callback) => {
      // this.$axios.post(`/api/verifyCode/verPicCode`, {picCode: value}).then(res => {
      //   callback()
      // }).catch((error) => {
      //   callback(new Error(error.response.data.message))
      // })
        this.$http({
              url: this.$http.adornUrl(`/api/verifyCode/verPicCode`),
              method: 'post',
              data: this.$http.adornData({picCode: value})
            }).then(res => {
              if(res.data.code==500)
              {
                   callback(new Error("验证码错误"))
                   return
              }
          callback()
      }).catch((error) => {
        callback(new Error(error.response.data.message))
      })
    }
    const validateImageCodeNew = (rule, value, callback) => {
      this.$http({
              url: this.$http.adornUrl(`/api/verifyCode/verPicCode`),
              method: 'post',
              data: this.$http.adornData({picCode: value})
            }).then(res => {
          if(res.data.code==500)
              {
                   callback(new Error("验证码错误"))
                   return
              }
          callback()
       }).catch((error) => {
         callback(new Error(error.response.data.message))
       })

      // this.$axios.post(`/api/verifyCode/verPicCode`, {picCode: value}).then(res => {
      //   callback()
      // }).catch((error) => {
      //   callback(new Error(error.response.data.message))
      // })
    }

    const validatePhoneCode = (rule, value, callback) => {

       let parms = {
        type: this.page.isChangeManage,
        phoneCode: value,
        phoneNum: this.page.isChangeManage === 4 ? this.page.newManager.phone : this.ruleForm.phone
      }
      if(value=="0000")
      {
       callback()
      }
      else{
         callback(new Error("手机验证码输入错误"))
      }

      // this.$axios.post(`/api/verifyCode/verPhoneCode`, {
      //   type: this.page.isChangeManage,
      //   phoneCode: value,
      //   phoneNum: this.page.isChangeManage === 4 ? this.page.newManager.phone : this.ruleForm.phone
      // }
      // ).then(res => {
      //   callback()
      // }).catch((error) => {
      //   callback(new Error(error.response.data.message))
      // })
    }

    return {
      ruleForm: {
        phone: '',
        tenant: '',
        password: '',
        imageCode: '',
        phoneCode: '',
        newimageCode: '',
        newphoneCode: ''
      },
      page: {
        isChangeManage: 3,
        accounts: [],
        newManager: '',
        manager: '',
        validateState: {
          phone: {},
          tenant: {},
          password: {},
          imageCode: {},
          phoneCode: {},
          newimageCode: {},
          newphoneCode: {}
        },
        submitFormState: {
          total: 3,
          showState: false,
          btnText: '重新登录 (3)',
          showIcon: '',
          showText: '',
          isError: 'isError',
          isSuccess: 'isSuccess',
          isActive: false
        }
      },
      rules: {
        phone: [
          {required: true, message: '请输入手机号', trigger: 'blur'},
          {message: '手机号不正确', trigger: 'blur', pattern: /^1[3|4|5|7|8][0-9]\d{8}$/}
        ],
        password: [
          {required: true, message: '请输入密码', trigger: 'blur'}
        ],
        imageCode: [
          {required: true, message: '请输入图片验证码', trigger: 'blur'},
          {validator: validateImageCode, trigger: 'blur'}
        ],
        phoneCode: [
          {required: true, message: '请输入手机验证码', trigger: 'blur'},
          {validator: validatePhoneCode, trigger: 'blur'}
        ],
        newimageCode: [
          {required: true, message: '请输入图片验证码', trigger: 'blur'},
          {validator: validateImageCodeNew, trigger: 'blur'}
        ],
        newphoneCode: [
          {validator: validatePhoneCode, trigger: 'blur'}
        ]
      }
    }
  },
  computed: {
    hasNewManage () {
      return Object.keys(this.page.newManager).length > 0
    }
  },
  mounted () {
    this.loadManager()
  },
  methods: {
    nextStep () {
      // 校验
      this.validate().catch(this.showErrors)
    },
    // 校验管理员
    checkCurrentManager () {
      let parms = {
        phone: this.ruleForm.phone,
        password: this.ruleForm.password,
        phoneCode: this.ruleForm.phoneCode,
        id: this.manager.userId
      }

     this.$http({
              url: this.$http.adornUrl(`/api/manage/checkCurrentManager`),
              method: 'post',
              data: this.$http.adornData(parms)
            }).then(({ res }) => {
              debugger
              this.page.isChangeManage = 4
              this.$refs['ruleForm'].resetFields()
            }).catch(error => {
        this.showError(error.field, error.message)
      })
      // this.$axios.post('/api/manage/checkCurrentManager', parms).then(res => {
      //   // 置空操作
      //   this.page.isChangeManage = 4
      //   this.$refs['ruleForm'].resetFields()
      // }).catch(error => {
      //   this.showError(error.field, error.message)
      // })
    },
    // 变更管理员
    transferManager () {
      let parms = {
        phone: this.page.newManager.phone,
        phoneCode: this.ruleForm.newphoneCode,
        id: this.page.newManager.id
      }
      this.$http({
              url: this.$http.adornUrl(`/api/manage/transferManager`),
              method: 'post',
              data: this.$http.adornData(parms)
            }).then(({ res }) => {
             this.page.submitFormState.showState = true
             this.changeSuccess()
            }).catch(error => {
       this.showError(error.field, error.message)
      })
      
      // this.$axios.post('/api/manage/transferManager', parms).then(res => {
      //   this.page.submitFormState.showState = true
      //   this.changeSuccess()
      // }).catch(error => {
      //   this.showError(error.field, error.message)
      // })
    },
    goPage () {
      if (this.page.submitFormState.isActive) {
        this.$http({
            url: this.$http.adornUrl('/sys/logout'),
            method: 'post',
            data: this.$http.adornData()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              clearLoginInfo()
              this.$router.push({ name: 'login' })
            }
          })
      }
    },
    changeSuccess () {
      this.page.submitFormState.showIcon = 'el-icon-success'
      this.page.submitFormState.showText = '设置成功!'
      this.page.submitFormState.isActive = true
      let setInterval = window.setInterval(() => {
        this.page.submitFormState.total--
        this.page.submitFormState.btnText = '重新登录 (' + this.page.submitFormState.total + ')'
        console.log(this.page.submitFormState.btnText)
        if (this.page.submitFormState.total <= 0) {
          window.clearInterval(setInterval)
          this.goPage()
        }
      }, 1000)
    },
    changeError () {
      this.page.submitFormState.showIcon = 'el-icon-error'
      this.page.submitFormState.showText = '操作失败，请返回重试!'
      this.page.submitFormState.btnText = '返回修改'
      this.page.submitFormState.isActive = false
    },
    loadManager () {
          this.$http({
          url: this.$http.adornUrl('/api/accounts/current'),
          method: 'get',
        }).then(res => {
        this.manager = res.data
         this.ruleForm.phone = res.data.phone
       }).catch((error) => {
         this.showError('phone', error.response.data.message)
       })
      // return this.$axios.get(`/api/accounts/current`).then(res => {
      //   this.manager = res.data
      //   this.ruleForm.phone = res.data.phone
      // }).catch((error) => {
      //   this.showError('phone', error.response.data.message)
      // })
    },
    validate () {
      return new Promise((resolve, reject) => {
        this.$refs['ruleForm'].validate((valid, targets) => {
          if (valid) {
            if (this.page.isChangeManage === 3) {
              this.checkCurrentManager()
            }
            if (this.page.isChangeManage === 4) {
              this.transferManager()
            }
            resolve()
          } else {
            reject(targets)
          }
        })
      })
    },
    validateImageCode () {
      let flag = ''
      if (this.page.isChangeManage === 3) {
        flag = 'imageCode'
      }
      if (this.page.isChangeManage === 4) {
        flag = 'newimageCode'
      }
      return new Promise((resolve) => {
        this.$refs.ruleForm.validateField(flag, (errorMessage, fields) => {
          if (errorMessage) {
            this.showErrors(fields)
          } else {
            resolve()
          }
        })
      })
    },
    showErrors (errors) {
      for (const key in errors) {
        if (errors.hasOwnProperty(key)) {
          const target = errors[key]
          for (const error of target) {
            this.showError(error.field, error.message)
          }
        }
      }
    },
    showError (key, message) {
      this.page.validateState[key] = {
        isValid: true,
        message: message
      }
      setTimeout(() => {
        this.page.validateState[key].isValid = false
      }, 3000)
    },
    querySearchAsync (queryString, callback) {
      if (queryString) {
          this.$http({
          url: this.$http.adornUrl('/api/manage/getAccountsByPhone'),
          method: 'get',
          params: {phone: queryString}
        }).then(res => {
           if (res.data) {
             this.page.accounts = res.data
             callback(this.page.accounts)
           }
         })
        // this.$axios.get(`/api/manage/getAccountsByPhone`, {params: {phone: queryString}}).then(res => {
        //   if (res.data) {
        //     this.page.accounts = res.data
        //     callback(this.page.accounts)
        //   }
        // })
      }
    },
    handleSelect (item) {
      this.page.keyword = item.phone
      this.page.newManager = item
    }
  }
}
</script>

<style lang="scss">
label{
  font-weight: 500;
}
  .box-card {
    height: 100%;

  .box-body {
    padding-left: 200px;
    margin-top: 30px;
  }

  .el-input__inner {
    width: 350px;
  }

  .el-input--small .el-input__inner {
    height: 36px;
    line-height: 36px;
  }

  .el-form-item {
    height: 75px;
  }

  .el-form-item--small.el-form-item {
    margin-bottom: 25px;
  }
  .userPhone {
    float: left;
  }

  .el-form-item--small.el-form-item {
    margin-bottom: 0px;
  }

  .el-form-item--mini.el-form-item, .el-form-item--small.el-form-item {
    margin-bottom: 0px;
  }

  /* 修改展示状态文字 */
  .isSuccess {
    color: #67c23a;
  }

  .isError {
    color: red;
  }

  .showTip {
    text-align: center;
    /* margin-top: 80px; */
  }

  .showIcq {
    font-size: 50px;
  }

  .showBtn {
    margin-top: 50px;
  }

  .tip {
    text-align: center;
    width: 350px;
    height: 350px;
  }
  }
</style>
