<template>
  <div class="uploadBox" @paste="handlePaste">
    <el-upload
      ref="uploadBtn"
      v-loading="ocrLoading"
      :file-list="fileList"
      :limit="limit"
      action=""
      drag
      :element-loading-text="ocrLoadingText"
    >
      <div class="el-upload__text">{{ props.uploadText }}</div>
    </el-upload>
    <div style="z-index: -1">
      <h3>识别结果：</h3>
      <pre>
        {{ ocrResultText }}
      </pre>
    </div>
  </div>
</template>
<script setup lang="ts">
import { ref, computed } from 'vue'
import { ElMessage } from 'element-plus'
import { createWorker } from 'tesseract.js'
import type { UploadProps } from 'element-plus'

export interface Props extends UploadProps {
  modelValue: string[]
  uploadText: string
  autoUpload: boolean
  limit: number
  disabled: boolean
  accept: string
}
const props = withDefaults(defineProps<Props>(), {
  uploadText: '将图片按 Ctrl + V 粘贴至此处',
  autoUpload: false,
  disabled: false,
  limit: 1,
  accept: 'image/*'
})
const emit = defineEmits(['update:modelValue'])
const fileList = computed({
  get() {
    return props.modelValue
  },
  set(value) {
    emit('update:modelValue', value)
  }
})
// const fileList = defineModel()
// console.log(fileList.value)

const ocrLoading = ref(false)
const ocrLoadingText = ref('')
const ocrResultText = ref('')
const uploadBtn = ref<HTMLDivElement | null>(null)
// const uploadImage = ref(null)

// function handleFocus(): void {
//   uploadImage?.value?.blur()
// }

async function handleOCRRecognize(file): Promise<void> {
  ocrResultText.value = ''
  ocrLoading.value = true
  ocrLoadingText.value = '正在识别中，请稍后...'
  const worker = await createWorker({
    logger: (m) => {
      if (m.status === 'recognizing text' && m.progress == 1) {
        ElMessage.success('图片识别完成')
        ocrLoading.value = false
      }
    },
    // 暂未实现离线访问
    // https://github.com/naptha/tessdata/tree/gh-pages/4.0.0
    // 配置本地资源路径
    workerPath: window.location.origin + '/tesseract.js-offline/tesseract.js/worker.min.js',
    corePath:
      window.location.origin + '/tesseract.js-offline/tesseract.js-core/tesseract-core.wasm.js',
    langPath: '/tesseract.js-offline/lang-data'
  })

  await worker.loadLanguage('chi_sim')
  await worker.initialize('chi_sim')
  const {
    data: { text }
  } = await worker.recognize(file)
  console.log(text, '')
  ocrResultText.value = text
  await worker.terminate()
}
function handlePaste(event): void {
  const items = (event.clipboardData || window.clipboardData).items
  // console.warn('handlePaste: ', items.length)
  //去除粘贴到div事件
  event.preventDefault()
  event.returnValue = false
  let file = null
  if (!items || items.length === 0) {
    ElMessage.error('当前不支持本地图片上传')
    return
  }
  // 搜索剪切板items
  for (let i = 0; i < items.length; i++) {
    if (items[i].type.includes('image')) {
      file = items[i].getAsFile()
      break
    }
  }
  if (!file) {
    ElMessage.error('粘贴内容非图片')
    return
  }
  // console.log('handlePaste file: ', file)
  handleOCRRecognize(file)
}
</script>
