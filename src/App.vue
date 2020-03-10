<template>
  <div id="app">
    <div class="img">
      <div class="img-box">
        <img v-if="imgurl" ref="image" alt="crop" :src="imgurl" width="200px">
      </div>
      <div class="preview-box">
        <div class="big">
          <img v-if="preview" :src="preview" alt="big preview">
        </div>
        <div class="normal">
          <img v-if="preview" :src="preview" alt="normal preview">
        </div>
        <div class="small">
          <img v-if="preview" :src="preview" alt="small preview">
        </div>
      </div>
    </div>
    <div class="operate">
      <button @click="onChoose">选择图片</button>
      <button @click="onDownload">下载裁剪图片</button>
    </div>
    <input style="display: none;" type="file" ref="uploads" accept="image/png, image/jpeg, image/gif, image/jpg" @change="onUploadImg($event)">
  </div>
</template>

<script>
import Cropper from 'cropperjs'
import 'cropperjs/dist/cropper.css'

export default {
  name: 'App',
  data() {
    return {
      options: {
        viewMode: 1,
        initialAspectRatio: 1,
        movable: false,
        zoomable: false,
        rotatable: false,
        scalable: false,
        guides: false,
        cropBoxResizable: false
      },
      cropper: null,
      imgurl: '',
      preview: '',
      isPreview: true
    }
  },
  methods: {
    init() {
      this.$nextTick(() => {
        const options = {
          crop: () => {
            this.onPreview()
          }
        }
        this.cropper = new Cropper(this.$refs.image, Object.assign(options, this.options))
      })
    },
    onPreview() {
      if (this.isPreview) {
        this.isPreview = false
        setTimeout(() => {
          this.isPreview = true
        }, 50)
      } else {
        return false
      }
      const canvas = this.cropper.getCroppedCanvas()
      this.preview = canvas.toDataURL()
    },
    onUploadImg(e) {
      const file = e.target.files[0]
      if (!/\.(gif|jpg|jpeg|png|bmp|GIF|JPG|PNG)$/.test(e.target.value)) {
        alert('图片类型必须是.gif,jpeg,jpg,png,bmp中的一种')
        return false
      }
      const reader = new FileReader()
      reader.onload = e => {
        if (typeof e.target.result === "object") {
          this.imgurl = window.URL.createObjectURL(new Blob([e.target.result]))
        } else {
          this.imgurl = e.target.result
        }
        this.init()
      }
      reader.readAsArrayBuffer(file)
    },
    onDownload() {
      const a = document.createElement('a')
      a.download = 'demo'
      a.href = this.preview
      document.body.appendChild(a)
      a.click()
      document.body.removeChild(a)
    },
    onChoose() {
      this.$refs.uploads.click()
    }
  }
}
</script>

<style lang="less">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  .img {
    display: flex;
    justify-content: center;
    .img-box {
      width: 400px;
      height: 400px;
      &:empty {
        background-color: rgb(247, 247, 247);
      }
      img {
        display: block;
        max-width: 100%;
        margin: 0 auto;
      }
    }
    .preview-box {
      margin-left: 40px;
      .big, .normal, .small {
        &:empty {
          background-color: rgb(247, 247, 247);
        }
        img {
          width: 100%;
          height: 100%;
        }
      }
      .big {
        width: 200px;
        height: 200px;
        margin-bottom: 25px;
      }
      .normal {
        width: 100px;
        height: 100px;
        margin-bottom: 25px;
      }
      .small {
        width: 50px;
        height: 50px;
      }
    }
  }
  .operate {
    margin-top: 15px;
    button {
      border-radius: 4px;
      padding: 2px 12px;
      outline: none;
    }
    button+button {
      margin-left: 20px;
    }
  }
}
</style>
