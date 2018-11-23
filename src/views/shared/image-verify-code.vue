<template>
  <div class="image-verify-code">
    <el-form-item :prop="propName || 'imageCode'">
      <div class="verify-input">
        <el-input :value="imageCode" @change="update" placeholder="请输入图片验证码" maxlength="4"/>
      </div>
      <div class="verify-button">
        <!-- <img   id="codeImg":src="imageSrc" @click="getCaptcha()" alt=""> -->
         <img
          id="codeImg"
          @click.stop="changeCode"
          alt="点击更换"
          title="点击更换"
          @click="changeCode"
          :src="imageSrc"
        > 
      </div>
    </el-form-item>
  </div>
</template>

<script>
import { getUUID } from '@/utils'
export default {
  name: 'image-verify-code',
  props: ['imageCode', 'propName'],
  data () {
    return {
      uuid:'',
      imageSrc: '',
      rules: {
        imageCode: [
          {required: true, message: '请输入手机号', trigger: 'blur'}
        ]
      }
    }
  },
  mounted () {
    //this.changeCode()
  },
  created () {
    this.changeCode()
  },
  methods: {
    changeCode () {
     this.$http({
            url: this.$http.adornUrl(`/api/verifyCode/generatePicCode?t=` + new Date().getTime()),
            method: 'get',
          }).then((res) => {
           this.imageSrc = 'data:image/jpg;base64,' + res.data
         })
    },
     // 获取验证码
    getCaptcha () {
      this.uuid = getUUID()
      this.imageSrc = this.$http.adornUrl(`/captcha.jpg?uuid=${this.uuid}`)
    },
    update (value) {
      this.$emit('input', value)
    }
  }
}
</script>

<style lang="scss">
  .image-verify-code {

  .verify-input, .verify-button {
    display: inline-block;
  }

  .verify-button {
    width: 114px;
    height: 35px;
    margin-left: 10px;
    padding-left: 20px;
    // padding: 5px 17.5px;
     border-radius: 4px;
     background-color: #e3e2f0;
     //vertical-align: middle;
  }

  }
</style>
