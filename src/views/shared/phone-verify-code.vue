<template>
  <el-form-item class="phone-verify-code" :prop="propName || 'phoneCode'">
    <div class="verify-input">
      <el-input :value="phoneCode" @change="update" placeholder="请输入短信验证码" maxlength="4"/>
    </div>
    <div class="verify-button">
      <el-button @click="sendCode" :disabled="btnDisabled" type="primary" round>{{ btnText }}</el-button>
    </div>
  </el-form-item>
</template>

<script>
export default {
  name: 'phone-verify-code',
  props: ['phone', 'phoneCode', 'imgCode', 'isFirstLogin', 'isChangeManage', 'propName', 'isRestPassword'],
  data () {
    return {
      time: 0
    }
  },
  methods: {
    sendCode () {
      let form = this
      do {
        form = form.$parent
      } while (!form.validateImageCode)

      form.validateImageCode().then(() => {
            this.$http({
              url: this.$http.adornUrl(`/api/verifyCode/sendPhoneCode`),
              method: 'post',
              data: this.$http.adornData({
          message: this.isChangeManage || this.isRestPassword ? '重置管理员' : (this.isFirstLogin ? '设置密码' : '登录密码'),
          picCode: this.imgCode,
          phoneNum: this.phone,
          type: this.isChangeManage ? this.isChangeManage : (this.isFirstLogin || this.isRestPassword ? '2' : '1')
           })
            })
        // // 执行发送验证码
        // this.$axios.post(`/api/verifyCode/sendPhoneCode`, {
        //   message: this.isChangeManage || this.isRestPassword ? '重置管理员' : (this.isFirstLogin ? '设置密码' : '登录密码'),
        //   picCode: this.imgCode,
        //   phoneNum: this.phone,
        //   type: this.isChangeManage ? this.isChangeManage : (this.isFirstLogin || this.isRestPassword ? '2' : '1')
        // }
        // )
        this.resetInterval()
      })
    },
    resetInterval () {
      this.time = 60

      let interval = setInterval(() => {
        if (this.time > 0) {
          this.time--
        } else {
          clearInterval(interval)
        }
      }, 1000)
    },
    update (value) {
      this.$emit('input', value)
    }
  },
  computed: {
    btnDisabled () {
      return this.time > 0
    },
    btnText: function () {
      return this.time > 0 ? this.time + 's 后重新获取' : '获取验证码'
    }
  }
}
</script>

<style lang="scss">
  .phone-verify-code {

  .verify-input, .verify-button {
    display: inline-block;
  }

  .verify-button {
    margin-left: 10px;

  button {
    width: 115px;
  }

  }
  }
</style>
