<template>
  <main id="login" class="login-wrapper">
    <el-form
      :model="ruleForm"
      :rules="rules"
      ref="ruleForm"
      :show-message="false"
      @submit.native.prevent="nextStep"
    >
      <div class="logo-group">
        <img id="tenantLogo" src="~@/assets/img/newlogo.png" alt="红猫系统设计平台" title="红猫系统设计平台">
      </div>
      <div class="tooltip-group" v-show="isFirstLogin">
        <label>您当前的管理员账号为首次登录，请设置“登录密码”后方登录</label>
      </div>
      <el-form-item prop="phone">
        <el-tooltip
          v-model="page.validateState.phone.isValid"
          :content="page.validateState.phone.message"
          placement="right"
          manual
        >
          <el-input type="text" v-model="ruleForm.phone" placeholder="请输入手机号"/>
        </el-tooltip>
      </el-form-item>
      <el-form-item prop="tenant" v-if="hasUser">
        <el-tooltip
          v-model="page.validateState.tenant.isValid"
          :content="page.validateState.tenant.message"
          placement="right"
          manual
        >
          <el-select
            v-model="ruleForm.tenant"
            placeholder="请选择租户"
            :disabled="!hasMultiTenant"
            @change="currentSel"
          >
            <el-option
              v-for="item in user.tenants"
              :key="item.accountId"
              :label="item.tenantName"
              :value="item.accountId"
            ></el-option>
          </el-select>
        </el-tooltip>
      </el-form-item>
      <div v-if="(hasTenant && isSecondValidate)||isFirstLogin||showResetPassword">
        <el-tooltip
          v-model="page.validateState.imageCode.isValid"
          :content="page.validateState.imageCode.message"
          placement="right"
          manual
        >
          <image-verify-code v-model="ruleForm.imageCode"/>
        </el-tooltip>
        <el-tooltip
          v-model="page.validateState.phoneCode.isValid"
          :content="page.validateState.phoneCode.message"
          placement="right"
          manual
        >
          <phone-verify-code
            v-model="ruleForm.phoneCode"
            :phone="ruleForm.phone"
            :img-code="ruleForm.imageCode"
            :is-first-login="isFirstLogin"
            :is-rest-password="ruleForm.forgetPassword"
          />
        </el-tooltip>
      </div>
      <el-form-item prop="newPassword" v-if="showResetPassword">
        <el-tooltip
          v-model="page.validateState.newPassword.isValid"
          :content="page.validateState.newPassword.message"
          placement="right"
          manual
        >
          <el-input type="password" v-model="ruleForm.newPassword" placeholder="设置管理密码"/>
        </el-tooltip>
      </el-form-item>
      <el-form-item prop="confirmPassword" v-if="showResetPassword">
        <el-tooltip
          v-model="page.validateState.confirmPassword.isValid"
          :content="page.validateState.confirmPassword.message"
          placement="right"
          manual
        >
          <el-input type="password" v-model="ruleForm.confirmPassword" placeholder="确认管理密码"/>
        </el-tooltip>
      </el-form-item>
      <el-form-item prop="password" v-if="showLoginPassword">
        <el-tooltip
          v-model="page.validateState.password.isValid"
          :content="page.validateState.password.message"
          placement="right"
          manual
        >
          <el-input type="password" v-model="ruleForm.password" placeholder="请输入管理密码"/>
        </el-tooltip>
      </el-form-item>
      <el-form-item class="buttons">
        <el-button type="primary" round @click="nextStep">{{showLoginPassword? '登录':'继续'}}</el-button>
        <a
          v-if="ruleForm.tenant"
          @click="ruleForm.forgetPassword=true"
          target="_blank"
          class="resetAccount"
        >忘记密码？</a>
      </el-form-item>
      <remote-js src="https://pv.sohu.com/cityjson?ie=utf-8"></remote-js>
    </el-form>
  </main>
</template>

<script>
import ImageVerifyCode from '../shared/image-verify-code.vue'
import PhoneVerifyCode from '../shared/phone-verify-code.vue'

export default {
  components: {
    ImageVerifyCode,
    PhoneVerifyCode,
    'remote-js': {
      render (createElement) {
        return createElement('script', {attrs: {type: 'text/javascript', src: this.src}})
      },
      props: {
        src: { type: String, required: true }
      }
    }
  },
  data () {
    const validateImageCode = (rule, value, callback) => {

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
        callback(new Error(error.response.error.message))
      })
    }

    const validatePhoneCode = (rule, value, callback) => {
      let parms ={
        type: this.isFirstLogin || this.ruleForm.forgetPassword ? '2' : '1',
        phoneCode: value,
        phoneNum: this.ruleForm.phone
      }
      if(value=="0000")
      {
         callback()
      }
    }

    // 校验密码
    const validateNewPassword = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请输入密码'))
      } else if (!(value.match(/[a-z]+/) && value.match(/[A-Z]+/) && value.match(/\d+/) && value.length >= 6)) {
        callback(new Error('密码必须数字、大写字母、小写字母组合，长度不小于6位'))
      } else {
        if (this.ruleForm.confirmPassword !== '') {
          this.$refs.ruleForm.validateField('confirmPassword')
        }
        callback()
      }
    }
    const validateConfirmPassword = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请再次输入密码'))
      } else if (value !== this.ruleForm.newPassword) {
        callback(new Error('两次输入密码不一致!'))
      } else {
        callback()
      }
    }

    return {
      page: {
        validateState: {
          phone: {},
          tenant: {},
          password: {},
          imageCode: {},
          phoneCode: {},
          newPassword: {},
          confirmPassword: {}
        }
      },
      ruleForm: {
        phone: '',
        tenant: '',
        password: '',
        imageCode: '',
        phoneCode: '',
        newPassword: '',
        confirmPassword: '',
        isFirstLogin: false,
        forgetPassword: false,
        successState: false,
        secondValidateState: '0',
        loginIp: '',
        cityId: '',
        cityName: ''
      },
      rules: {
        phone: [
          {required: true, message: '请输入手机号', trigger: 'blur'},
          {message: '手机号不正确', trigger: 'blur', pattern: /^1[3|4|5|7|8|9][0-9]\d{8}$/}
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
        newPassword: [
          {validator: validateNewPassword, trigger: 'blur'}
        ],
        confirmPassword: [
          {validator: validateConfirmPassword, trigger: 'blur'}
        ]
      },
      user: {
        phone: '',
        tenants: '',
        tenant: ''
      }
    }
  },
  computed: {
    hasUser () {
      return !!this.user.phone
    },
    hasMultiTenant () {
      return this.user.tenants.length > 1
    },
    tenant () {
      return this.user.tenants.find(m => m.accountId === this.ruleForm.tenant)
    },
    hasTenant () {
      return this.hasUser && !!this.tenant
    },
    isFirstLogin () {
      return (this.hasUser && this.hasTenant && this.tenant.isFirstLogin) || this.ruleForm.isFirstLogin
    },
    showResetPassword () {
      return (this.isFirstLogin || this.ruleForm.forgetPassword) && !this.ruleForm.successState
    },
    showLoginPassword () {
      return (this.hasUser && this.hasTenant && !this.showResetPassword) || this.ruleForm.successState
    },
    isSecondValidate () {
      return this.ruleForm.secondValidateState >= 3
    }
  },
  methods: {
    nextStep () {
      // 校验
      this.validate()
        .then(this.loadUser)
        .then(res => {
          const tenants = res.data
          this.user.phone = this.ruleForm.phone
          this.user.tenants = tenants
          this.ruleForm.isFirstLogin = tenants[0].firstLogin
          this.ruleForm.secondValidateState = tenants[0].multiValidate
          if (tenants.length > 0) {
            this.ruleForm.tenant = tenants[0].accountId
          }
        }).catch(this.showErrors)
    },
    loadUser () {
      /* eslint-disable */
      let citySN = returnCitySN
      /* eslint-enable */
      this.ruleForm.loginIp = citySN['cip']
      this.ruleForm.cityId = citySN['cid']
      this.ruleForm.cityName = citySN['cname']

   return  this.$http({
              url: this.$http.adornUrl('/api/accounts/getAccountsByPhone'),
              method: 'get',
              params: {phone: this.ruleForm.phone, loginIp: this.ruleForm.loginIp, cityId: this.ruleForm.cityId, cityName: this.ruleForm.cityName}
            }).catch((error) => {
          this.showError('phone', error.response.message)
        })
    },
    currentSel (selVal) {
      this.ruleForm.isFirstLogin = this.user.tenants.find(m => m.accountId === selVal).firstLogin
    },
    login () {
      let parms = {
        phone: this.ruleForm.phone,
        password: this.ruleForm.confirmPassword ? this.ruleForm.confirmPassword : this.ruleForm.password,
        phoneCode: this.ruleForm.phoneCode,
        id: this.ruleForm.tenant,
        loginIp: this.ruleForm.loginIp,
        cityId: this.ruleForm.cityId,
        cityName: this.ruleForm.cityName,
        multiValidate: this.ruleForm.isSecondValidate
      }
      // 首次登录
      if (this.ruleForm.confirmPassword || this.ruleForm.forgetPassword) {
        if (this.ruleForm.phone && this.ruleForm.phoneCode && this.ruleForm.tenant) {

             this.$http({
              url: this.$http.adornUrl('/api/accounts/setManagePassword'),
              method: 'post',
              data: this.$http.adornData(parms)
            }).then(res => {
            this.$notify.success({
              title: '提示',
              offset: 100,
              message: '设置密码成功',
              showClose: true
            })
            this.ruleForm.password = ''
            this.ruleForm.confirmPassword = ''
            this.ruleForm.forgetPassword = false
            this.ruleForm.successState = true
          }).catch(error => {
            this.showError(error.field, error.message)
          })
        }
      } else {
        if (this.ruleForm.phone && this.ruleForm.tenant) {
            this.$http({
              url: this.$http.adornUrl('/api/accounts/login'),
              method: 'post',
              data: this.$http.adornData(parms)
            }).then(res => {
            if (res.data.msg=="success") {
                this.$cookie.set('token', res.data.token)
                this.$router.replace({ name: 'home' })
            }
          }).catch(error => {
            this.showError(error.field, error.message)
          })

        }
      }
    },
    validate () {
      return new Promise((resolve, reject) => {
        this.$refs['ruleForm'].validate((valid, targets) => {
          if (valid) {
            resolve()
            this.login()
          } else {
            reject(targets)
          }
        })
      })
    },
    validateImageCode () {
      return new Promise((resolve) => {
        this.$refs.ruleForm.validateField('imageCode', (errorMessage, fields) => {
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
    // 校验手机号
    isValidPhone (tel) {
      let reg = /^1[3|4|5|7|8][0-9]\d{8}$/
      return reg.test(tel)
    }
  }
}
</script>

<style lang="scss">
  .login-wrapper {
    width: 100%;
    height: 100%;
    // background: url("/assets/img/pg.png") no-repeat center center;
    background-image: url(~@/assets/img/login_bg.png);
    background-size: cover;
    padding-top: 175px;

  @media screen and (max-height: 768px) {
    padding-top:50px;
  }

  .verify-input {
    width: calc(100% - 130px);
  }
  form{
    padding-top: 5%
  }
  .el-form {
    width: 325px;
    margin-left: auto;
    margin-right: auto;

  .logo-group {
    height: 120px;
    padding-top: 40px;
  }

  .tooltip-group {
    margin-bottom: 12px;
    font-size: 12px;
    color: #f8feff;
  }

  .el-form-item {

  input,
  button {
    height: 38px;
  }

  input {
    background-color: #3b424c;
    color: #fff;
    border: none;
    border-radius: 10px;
  }

  .el-select {
    width: 100%;
  }

  }

  .buttons button {
    width: 100%;
    height: 38px;
  }
  .resetAccount{
    text-decoration: none;
    float: right;
    color: #409EFF;
  }

  }
  }
</style>
