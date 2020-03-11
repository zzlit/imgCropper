<template>
  <div class="upload">
    <div :id="id" style="width: 100%; height: 100%;" />
    <img class="upload-img" ref="uploadimg" :src="imgurl" alt="upload" v-if="imgurl">
  </div>
</template>

<script>
import plupload from 'plupload'

export default {
  name: 'Upload',
  props: {
    id: {
      type: String,
      default: 'UPLOAD'
    }
  },
  data () {
    return {
      alioss: {},
      options: {
        runtimes: 'html5, flash, silverlight, html4',
        multi_selection: false,
        flash_swf_url: '/plupload/js/Moxie.swf',
        silverlight_xap_url: '/plupload/js/Moxie.xap'
      },
      plupload: null,
      imgurl: '', // 预览的图片地址 base64 or blob
      uploadImgurl: ''
    }
  },
  methods: {
    getOssConfig () {
      const url = /* aliossapi + */ 'app/cdn/token?dir=admin/icon_img'
      this.$ajax.get(url).then(({data: {data, code}}) => {
        /*
        * 处理相关操作
        */
        this.alioss = data
        this.initFrontUpload()
      })
    },
    initFrontUpload () {
      const options = Object.assign(
        {
          browse_button: this.id,
          url: this.alioss.host,
          init: {
            BeforeUpload: this.beforeUpload,
            FilesAdded: this.filesAdded,
            FileUploaded: this.fileUploaded
          }
        },
        this.options
      )
      this.plupload = new plupload.Uploader(options)
      this.plupload.init()
    },
    beforeUpload (up, files) {
      this.plupload.setOption({
        multipart_params: {
          key: this.alioss.dir + '/' + this.uploadImgurl,
          policy: this.alioss.policy,
          OSSAccessKeyId: this.alioss.accessid,
          success_action_status: 200,
          signature: this.alioss.signature
        }
      })
    },
    filesAdded (up, files) {
      // moxie api: https://github.com/moxiecode/moxie/wiki/API
      // https://github.com/moxiecode/plupload/issues/1469#issuecomment-320438490
      const moxie = plupload.moxie
      const file = files[0]

      const suffix = files[0].name.split('.').pop() // 文件类型
      const timeStamp = new Date().getTime() // 时间戳
      const random = Math.floor(Math.random() * Math.floor(10000000)) // 随机数
      this.uploadImgurl = `${timeStamp}_${random}.${suffix}`
      if (file.type === 'image/gif') {
        let fr = new moxie.file.FileReader()
        fr.onload = () => {
          this.imgurl = fr.result
          fr.destroy()
          fr = null
        }
        fr.readAsDataURL(file.getSource())
      } else {
        let preview = new moxie.image.Image()
        preview.onload = () => {
          preview.resize({
            width: 100,
            fit: false
          })
          preview.crop(100, 100)
          this.imgurl = preview.type === 'image/jpeg' ? preview.getAsDataURL('image/jpeg', 80) : preview.getAsDataURL()
          preview.destroy()
          preview = null
        }
        preview.load(file.getSource())
      }
    },
    fileUploaded (up, file, response) {
      const image = new Image()
      image.src = this.alioss.dir + '/' + this.uploadImgurl
      image.onload = () => {
        console.log(image.width, image.height)
      }
      // this.$emit('success', response.status === 200 ? this.uploadImgurl : null)
    }
  }
}
</script>

<style scoped>
.upload {
  position: relative;
  width: 100px;
  height: 100px;
  background-color: #ececec;
  cursor: pointer;
}
.upload-img {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
}
</style>
