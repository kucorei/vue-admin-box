<template>
  <Layer :layer="layer" width="1000">
    <div class="container">
      <div class="box">
        <img :id="imgId" class="default-img" :src="modelValue" />
      </div>
      <div class="box">
        <div :class="previewClass" class="preview">
        </div>
      </div>
    </div>
    <Tip :cropper="cropper" v-if="cropper" />
    <div class="handle-box">
      <el-upload
        class="upload-demo"
        action=""
        :auto-upload="false"
        accept="image/*"
        ref="uploadDom"
        :on-change="uploadchange"
      >
        <el-button type="primary" icon="el-icon-upload" style="margin-right: 10px;">上传图片</el-button>
      </el-upload>
      <el-button type="primary" icon="el-icon-download" @click="downloadCropper">下载截图至本地</el-button>
      <el-button type="primary" icon="el-icon-picture" @click="saveAsModel">保存截图</el-button>
    </div>
  </Layer>
</template>

<script lang="ts">
import { defineComponent, nextTick, watch, ref } from 'vue'
import Layer from '@/components/layer/index.vue'
import Tip from './tip.vue'
import Cropper from 'cropperjs'
import 'cropperjs/dist/cropper.css'
interface Command {
  
}
export default defineComponent({
  components: {
    Layer,
    Tip
  },
  props: {
    layer: {
      type: Object,
      required: true,
      deafult: () => {
        return {
          show: false
        }
      }
    },
    modelValue: {
      type: String,
      default: ''
    }
  },
  setup(props, context) {
    let cropper = ref(null as Cropper|null)
    const imgId = 'image' + (new Date()).getTime()
    const previewClass = 'preview' + (new Date()).getTime()
    let uploadDom = ref(null as any|null)
    watch(() => props.layer.show, (newVal) => {
      if (!newVal) {
        return
      }
      nextTick(() => {
        initCropper()
      })
    })
    // init the cropperjs
    function initCropper() {
      if (cropper.value) {
        cropper.value.destroy()
      }
      const image = document.getElementById(imgId) as HTMLImageElement
      cropper.value = new Cropper(image, {
        preview: '.' + previewClass
      });
    }
    // 文件选择监控
    function uploadchange(file: any, fileList: any) {
      file2base64(file)
      uploadDom.value.clearFiles()
    }
    // 文件对象转base64
    function file2base64(file: any) {
      let imgFile = new FileReader()
      imgFile.readAsDataURL(file.raw)
      imgFile.onload = function(e) {
        const image = document.getElementById(imgId) as HTMLImageElement
        image.setAttribute('src', this.result as string)
        initCropper()
      }
    }
    // 下载本地图片
    function downloadCropper() {
      const canvas = cropper.value.getCroppedCanvas({
        maxWidth: 4096,
        maxHeight: 4096
      })
      const base64 = canvas.toDataURL() || ''
      let a = document.createElement('a')
      a.href = base64
      a.download = "截图下载.png"
      a.click()
    }
    // 保存至v-model数据,与下载本地图片类似，但需要执行数据上传操作
    function saveAsModel() {
      // 1. 获取base64数据
      const canvas = cropper.value.getCroppedCanvas({
        maxWidth: 4096,
        maxHeight: 4096
      })
      const base64 = canvas.toDataURL()
      // 2. 进行图片上传，并拿到上传后的地址，暂时省略
      // 3. 赋值给v-model
      context.emit('update:modelValue', base64)
      // 4. 关闭当前弹窗
      props.layer.show = false
    }
    return {
      cropper,
      uploadchange,
      imgId,
      previewClass,
      uploadDom,
      downloadCropper,
      saveAsModel
    }
  }
})
</script>

<style lang="scss" scoped>
  .container {
    display: flex;
    justify-content: space-between;
    height: 300px;
    overflow: hidden;
    .box {
      width: 49%;
      height: 100%;
      .default-img {
        display: block;
        max-width: 100%;
        height: 100%;
        margin: 0 auto;
      }
      .preview{
        width: 100%;
        height: 100%;
        overflow: hidden;
        background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQAQMAAAAlPW0iAAAAA3NCSVQICAjb4U/gAAAABlBMVEXMzMz////TjRV2AAAACXBIWXMAAArrAAAK6wGCiw1aAAAAHHRFWHRTb2Z0d2FyZQBBZG9iZSBGaXJld29ya3MgQ1M26LyyjAAAABFJREFUCJlj+M/AgBVhF/0PAH6/D/HkDxOGAAAAAElFTkSuQmCC');
        img {
          max-width: 100%;
        }
      }
    }
    
  }
  .handle-box {
    margin-top: 10px;
    display: flex;
    align-items: center;
    width: 49%;
  }
</style>