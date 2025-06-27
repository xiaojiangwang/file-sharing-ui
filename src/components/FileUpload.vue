<template>
  <div class="file-upload-container">
    <!-- 文件上传卡片 -->
    <a-card class="upload-card" :bordered="false">
      <a-upload-dragger
        name="file"
        action="/api/files/upload"
        :multiple="true"
        @success="handleUploadSuccess"
        @error="handleUploadError"
        :before-upload="beforeUpload"
        :class="{ 'upload-error': uploadStatus === 'error' }"
        :showUploadList="false"
      >
        <p class="ant-upload-drag-icon">
          <inbox-outlined />
        </p>
        <p class="ant-upload-text">点击或拖拽文件到此区域上传</p>
        <p class="ant-upload-hint">
          支持单个或批量上传，单个文件大小不超过100MB
        </p>
      </a-upload-dragger>
    </a-card>

    <!-- 文本上传卡片 -->
    <a-card class="text-upload-card" :bordered="false">
      <template #title>
        <span class="card-title">
          <form-outlined /> 文本上传
        </span>
      </template>
      <a-textarea
        v-model:value="textContent"
        placeholder="请输入要上传的文本内容"
        :rows="4"
        :maxLength="1000"
        show-count
      />
      <div class="text-upload-actions">
        <a-button type="primary" @click="uploadText" :loading="textUploading">
          <template #icon><upload-outlined /></template>
          上传文本
        </a-button>
      </div>
    </a-card>

    <!-- 文件列表卡片 -->
    <!-- 文本列表卡片 -->
    <a-card v-if="textList.length > 0" class="text-list-card" :bordered="false">
      <template #title>
        <span class="card-title">
          <form-outlined /> 已上传文本
        </span>
      </template>
      <a-list
        :data-source="textList"
        :pagination="{ pageSize: 5 }"
      >
        <template #renderItem="{ item }">
          <a-list-item>
            <template #actions>
              <a-space>
                <a-button type="link" @click="copyText(item)">
                  <template #icon><copy-outlined /></template>
                  复制
                </a-button>
                <a-button type="link" danger @click="confirmDeleteText(item)">
                  <template #icon><delete-outlined /></template>
                  删除
                </a-button>
              </a-space>
            </template>
            <a-list-item-meta>
              <template #description>
                <span>上传时间：{{ item.uploadTime }}</span>
              </template>
            </a-list-item-meta>
            <div class="text-content">{{ item.content }}</div>
          </a-list-item>
        </template>
      </a-list>
    </a-card>

    <!-- 文件列表卡片 -->
    <a-card v-if="fileList.length > 0" class="file-list-card" :bordered="false">
      <template #title>
        <span class="card-title">
          <folder-outlined /> 已上传文件
        </span>
      </template>
      <a-table
        :columns="columns"
        :data-source="fileList"
        :pagination="{ pageSize: 10 }"
        :row-key="record => record.fileName"
      >

        <template #bodyCell="{ column, record }">
          <template v-if="column.key === 'fileName'">
            <file-outlined /> {{ record.fileName }}
          </template>
          <template v-if="column.key === 'fileSize'">
            {{ formatFileSize(record.fileSize) }}
          </template>
          <template v-if="column.key === 'action'">
            <a-space>
              <a-button type="link" @click="downloadFile(record)">
                <template #icon><download-outlined /></template>
                下载
              </a-button>
              <a-button type="link" danger @click="confirmDelete(record)">
                <template #icon><delete-outlined /></template>
                删除
              </a-button>
            </a-space>
          </template>
        </template>
      </a-table>
    </a-card>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

// 获取文件列表
const fetchFileList = async () => {
  try {
    const response = await axios.get('/api/files')
    fileList.value = response.data.map(file => ({
      id: file.url.split('/').pop(), // 从URL中提取文件ID
      fileName: file.fileName,
      fileSize: file.size,
      uploadTime: new Date().toLocaleString() // 使用当前时间作为默认值
    }))
  } catch (error) {
    message.error('获取文件列表失败')
  }
}

// 组件挂载时获取文件列表
onMounted(() => {
  fetchFileList()
  fetchTextList()
})
import {
  InboxOutlined,
  FolderOutlined,
  FileOutlined,
  DownloadOutlined,
  DeleteOutlined,
  FormOutlined,
  UploadOutlined,
  CopyOutlined
} from '@ant-design/icons-vue'
import { message, Modal } from 'ant-design-vue'
import axios from 'axios'

const fileList = ref([])
const textList = ref([])
const uploadStatus = ref('normal')
const textContent = ref('')
const textUploading = ref(false)

// 获取文本列表
const fetchTextList = async () => {
  try {
    const response = await axios.get('/api/texts')
    textList.value = response.data.map(text => ({
      id: text.id,
      content: text.content,
      uploadTime: new Date().toLocaleString()
    }))
  } catch (error) {
    message.error('获取文本列表失败')
  }
}

// 上传文本
const uploadText = async () => {
  if (!textContent.value.trim()) {
    message.warning('请输入文本内容')
    return
  }

  textUploading.value = true
  try {
    const response = await axios.post('/api/texts/upload', textContent.value, {
      headers: {
        'Content-Type': 'text/plain'
      }
    })
    
    const newText = {
      id: response.data.id,
      content: response.data.content,
      uploadTime: new Date().toLocaleString()
    }
    textList.value.unshift(newText)
    textContent.value = ''
    message.success('文本上传成功')
  } catch (error) {
    message.error('文本上传失败')
  } finally {
    textUploading.value = false
  }
}

// 复制文本
const copyText = async (text) => {
  try {
    await navigator.clipboard.writeText(text.content)
    message.success('文本已复制到剪贴板')
  } catch (error) {
    message.error('复制失败')
  }
}

// 删除文本
const confirmDeleteText = (text) => {
  Modal.confirm({
    title: '确认删除',
    content: '确定要删除这条文本吗？',
    okText: '确认',
    cancelText: '取消',
    async onOk() {
      try {
        await axios.delete(`/api/texts/${text.id}`)
        textList.value = textList.value.filter(item => item.id !== text.id)
        message.success('文本删除成功')
      } catch (error) {
        message.error('文本删除失败')
      }
    }
  })
}

const columns = [
  {
    title: '文件名',
    dataIndex: 'fileName',
    key: 'fileName',
    ellipsis: true
  },
  {
    title: '大小',
    dataIndex: 'fileSize',
    key: 'fileSize',
    width: 120
  },
  {
    title: '上传时间',
    dataIndex: 'uploadTime',
    key: 'uploadTime',
    width: 180
  },
  {
    title: '操作',
    key: 'action',
    width: 180,
    align: 'center'
  }
]

const handleUploadSuccess = (response, file) => {
  uploadStatus.value = 'success'
  message.success('文件上传成功')
  // 从响应中提取文件ID
  const fileId = response.split('/').pop()
  const newFile = {
    id: fileId,
    fileName: file.name,
    fileSize: file.size,
    uploadTime: new Date().toLocaleString()
  }
  fileList.value.unshift(newFile)
  // 重置上传状态
  setTimeout(() => {
    uploadStatus.value = 'normal'
  }, 1000)
}

const handleUploadError = () => {
  uploadStatus.value = 'error'
  message.error('文件上传失败')
}

const beforeUpload = (file) => {
  const maxSize = 200 * 1024 * 1024 // 200MB
  if (file.size > maxSize) {
    message.error('文件大小不能超过200MB')
    return false
  }
  return true
}

const formatFileSize = (bytes) => {
  if (bytes === 0) return '0 B'
  const k = 1024
  const sizes = ['B', 'KB', 'MB', 'GB', 'TB']
  const i = Math.floor(Math.log(bytes) / Math.log(k))
  return (bytes / Math.pow(k, i)).toFixed(2) + ' ' + sizes[i]
}

const downloadFile = async (file) => {
  try {
    const response = await axios({
      url: `/api/files/${file.id}`,
      method: 'GET',
      responseType: 'blob'
    })
    const url = window.URL.createObjectURL(new Blob([response.data]))
    const link = document.createElement('a')
    link.href = url
    link.setAttribute('download', file.fileName)
    document.body.appendChild(link)
    link.click()
    document.body.removeChild(link)
  } catch (error) {
    message.error('文件下载失败')
  }
}

const confirmDelete = (file) => {
  Modal.confirm({
    title: '确认删除',
    content: `确定要删除文件 ${file.fileName} 吗？`,
    okText: '确认',
    cancelText: '取消',
    async onOk() {
      try {
        await axios.delete(`/api/files/${file.id}`)
        fileList.value = fileList.value.filter(item => item.id !== file.id)
        message.success('文件删除成功')
      } catch (error) {
        message.error('文件删除失败')
      }
    }
  })
}
</script>

<style scoped>
.file-upload-container {
  width: 100%;
}

.upload-card,
.text-upload-card {
  margin-bottom: 24px;
  background: #fafafa;
}

:deep(.upload-error .ant-upload-list-item-progress .ant-progress-bg) {
  background-color: #ff4d4f !important;
}

:deep(.ant-upload-list-item-error .ant-progress-bg) {
  background-color: #ff4d4f !important;
}

.file-list-card {
  background: #fff;
}

:deep(.ant-table-row) {
  height: 60px;
}

:deep(.ant-table-cell) {
  border: 1px solid #f0f0f0;
}

.card-title {
  font-size: 16px;
  font-weight: 500;
}

:deep(.ant-upload-drag) {
  padding: 48px;
}

.ant-upload-drag-icon {
  font-size: 48px;
  color: #40a9ff;
}

.ant-upload-text {
  font-size: 16px;
  margin: 16px 0;
}

.text-upload-actions {
  margin-top: 16px;
  display: flex;
  justify-content: flex-end;
}

.text-list-card {
  margin-top: 24px;
  background: #fff;
}

.text-content {
  max-height: 100px;
  overflow-y: auto;
  white-space: pre-wrap;
  word-break: break-all;
}

.text-actions {
  display: flex;
  justify-content: flex-end;
  gap: 8px;
}

.ant-upload-hint {
  color: rgba(0, 0, 0, 0.45);
}

:deep(.ant-table-pagination) {
  margin-bottom: 0 !important;
}
</style>