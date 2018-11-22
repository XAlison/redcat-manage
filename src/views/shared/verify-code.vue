<template>
    <div class="verify-code">
      <image-verify-code @validate="imageValidate" />
      <phone-verify-code :phone="phone" :disabled="imageValid" disabled-message="图片验证码失败" />
    </div>
</template>

<script>
import ImageVerifyCode from './image-verify-code.vue'
import PhoneVerifyCode from './phone-verify-code.vue'

export default {
  name: 'verify-code',
  props: ['phone'],
  data () {
    return {
      imageValid: false,
      phoneValid: false
    }
  },
  components: {
    ImageVerifyCode,
    PhoneVerifyCode
  },
  methods: {
    imageValidate (value) {
      this.imageValid = value
    }
  }
}
</script>

<style lang="scss">
.verify-code {

  > div {
    margin-bottom: 15px;
  }
}
</style>
