<template>
  <el-card class="box-card">
    <div slot="header" class="clearfix">
      <span>基本设置</span>
    </div>
    <div class="text item">
      <el-form label-width="80px">
        <el-form-item label="系统名称" prop="password">
          <el-input type="text" v-model="name" clearable v-bind:disabled="disabled"></el-input>
        </el-form-item>
        <el-form-item label="系统logo" prop="imgLogo">
          <el-upload
            ref="upload"
            action="/tenant/createTenant"
            class="avatar-uploader"
            :on-change="handleChange"
            :auto-upload="false"
            :show-file-list="false"
            :disabled="disabled"
          >
            <img id="imgLogo" v-if="imgLogo" :src="imgLogo" class="logo">
            <i v-else class="el-icon-plus logo-uploader-icon"></i>
          </el-upload>
          <p class="text_tips">建议尺寸：128*128</p>
        </el-form-item>
      </el-form>
      <div class="btn-group">
        <el-button type="primary" @click.native="changeInfo()" v-show="disabled">修改</el-button>
        <el-button type="primary" @click.native="submit()" v-show="!disabled">保存</el-button>
      </div>
    </div>
  </el-card>
</template>
<script>
export default {
  name: 'tenant-Info',
  data () {
    return {
      disabled: true,
      name: '',
      imgLogo: '',
      editable: false,
      imgSrc: null,
      id: ''
    }
  },
  mounted () {
    this.loadData()
  },
  methods: {
    loadData () {

      //     let _this = this
      // return this.$axios.get('/manage-api/tenant/getTenantInfoByTenantId').then((res) => {
      //   if (res.data) {
      //     _this.name = res.data.tenantName
      //     if (res.data.logo) {
      //       _this.logo = '/manage-api/tenant' + res.data.logo
      //     }
      //     _this.id = res.data.id
      //   }
      // })
      let _this = this
      this.$http({
            url: this.$http.adornUrl('/api/tenant/getTenantInfoByTenantId'),
            method: 'get',
          }).then((res) => {
         if (res.data) {
           _this.name = res.data.tenantName
           if (res.data.logo) {
             _this.imgLogo = res.data.logo
           }
           _this.id = res.data.id
         }
       })

    },
    changeInfo () {
      this.$refs.upload.clearFiles()
      this.disabled = false
    },
    handleChange (file, fileList) {
      this.imgLogo = URL.createObjectURL(file.raw)
    },
    submit () {
      if (this._checkForm()) {
        let _self = this
        let formData = new FormData()
        if (this.$refs.upload.uploadFiles[0]) {
          // 将文件转换为file
          let img = new Image()
          img.src = document.getElementById('imgLogo').src
          let imgName = this.$refs.upload.uploadFiles[0].name
          let base64 = _self._image2Base64(img)
          formData.append('file', _self._convertBase64UrlToBlob(base64), imgName)
        }
        formData.append('tenantName', this.name)
        formData.append('id', this.id)

         this.$http({
              url: this.$http.adornUrl(`/api/tenant/updateTenant`),
              method: 'post',
              data: formData
            }).then(() => {
          this.$message.info('保存成功!')
          })
          this.disabled = true


        // this.$axios.post('/api/tenant/updateTenant', formData).then(() => {
        //   this.$message.info('保存成功!')
        // })
        // this.disabled = true
      }
    },
    /**
       * 表单内容校验
       * @returns {boolean} 校验结果
       */
    _checkForm () {
      let result = true
      if (this.$refs.upload.uploadFiles[0] && !this.$refs.upload.uploadFiles[0].raw.type.startsWith('image/')) {
        result = false
        this.$message.error('你选择的不是jpg图片!')
      } else if (this.name.length === 0) {
        this.$message.error('请输入租户名!')
        result = false
      }

      return result
    },
    /**
       * 图片转base64
       * @param img 图片
       * @returns {string} 转换后的base64字符串
       * @private
       */
    _image2Base64 (img) {
      let canvas = document.createElement('canvas')
      canvas.width = img.width
      canvas.height = img.height
      let ctx = canvas.getContext('2d')
      ctx.drawImage(img, 0, 0, img.width, img.height)
      let dataURL = canvas.toDataURL('image/jpeg')
      return dataURL
    },
    /**
       * base64 转blob
       * @param urlData  转化后的base64
       * @returns {Blob} blob对象
       * @private
       */
    _convertBase64UrlToBlob (urlData) {
      let bytes = window.atob(urlData.split(',')[1])
      let ab = new ArrayBuffer(bytes.length)
      let ia = new Uint8Array(ab)
      for (let i = 0; i < bytes.length; i++) {
        ia[i] = bytes.charCodeAt(i)
      }
      return new Blob([ab], {type: 'image/jpeg'})
    }
  }
}
</script>

<style lang="scss" scoped>
  .el-table__body-wrapper {
    min-height: 200px;
  }
label{
  font-weight: 500;
}
  .logo-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }

  .logo-uploader .el-upload:hover {
    border-color: #409eff;
  }

  .logo-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 150px;
    height: 150px;
    line-height: 150px;
    text-align: center;
  }

  .logo {
    width: 150px;
    height: 150px;
    display: block;
  }

  .btn-group {
    padding-left: 100px;
    padding-top: 150px;
  }

  .el-input {
    width: 380px;
  }

  .container {
    padding-left: 50px;
    padding-top: 30px;
    .title {
      padding-bottom: 50px;
      span {
        font-size: 30px;
      }
    }
  }
  .text {
    font-size: 14px;
  }

  .item {
    margin-bottom: 18px;
  }

  .clearfix:before,
  .clearfix:after {
    display: table;
    content: "";
  }
  .clearfix:after {
    clear: both
  }

  .box-card {
    height: 500px;
    width: 100%;
  }
</style>