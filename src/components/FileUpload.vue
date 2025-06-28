<template>
  <div class="file-upload-container">
    <a-tabs v-model:activeKey="activeTab">
      <!-- 文件上传标签页 -->
      <a-tab-pane key="file">
        <template #tab>
          <span>
            <folder-outlined />
            文件上传
          </span>
        </template>
        <!-- 文件上传卡片 -->
        <a-card class="upload-card" :bordered="false">
          <a-upload-dragger
            name="file"
            action="/api/files/upload"
            :multiple="true"
            :customRequest="customRequest"
            :before-upload="beforeUpload"
            :class="{ 'upload-error': uploadStatus === 'error' }"
            :showUploadList="false"
          >
            <p class="ant-upload-drag-icon">
              <inbox-outlined />
            </p>
            <p class="ant-upload-text">点击或拖拽文件到此区域上传</p>
            <p class="ant-upload-hint">
              支持单个或批量上传，单个文件大小不超过{{ maxFileSize }}MB
            </p>
          </a-upload-dragger>
          
          <!-- 密码输入框 -->
          <div class="upload-options">
            <a-input-password
              v-model:value="filePassword"
              placeholder="可选：设置文件密码保护"
              :maxLength="50"
              allow-clear
            />
            <a-input
              v-model:value="fileRemark"
              placeholder="可选：添加备注信息"
              :maxLength="100"
              allow-clear
            />
          </div>
          
          <!-- 上传进度条 -->
          <div v-if="uploadProgress > 0 && uploadProgress < 100" class="upload-progress">
            <a-progress :percent="uploadProgress" :show-info="false" status="active" size="small" />
          </div>
        </a-card>

        <!-- 文件列表卡片 -->
        <a-card v-if="fileList.length > 0" class="file-list-card" :bordered="false">
          <template #title>
            <span class="card-title">
              <folder-outlined /> 已上传文件 ({{ fileList.length }})
            </span>
          </template>
          <a-table
            :columns="columns"
            :data-source="fileList"
            :pagination="{ pageSize: 5 }"
            :row-key="record => record.id"
            size="small"
          >
            <template #bodyCell="{ column, record }">
              <template v-if="column.key === 'fileName'">
                <file-outlined /> {{ record.fileName }}
                <lock-outlined v-if="record.passwordProtected" style="margin-left: 8px; color: #faad14;" />
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
      </a-tab-pane>

      <!-- 文本上传标签页 -->
      <a-tab-pane key="text">
        <template #tab>
          <span>
            <form-outlined />
            文本上传
          </span>
        </template>
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
          
          <!-- 密码和备注输入框 -->
          <div class="text-upload-options">
            <a-input-password
              v-model:value="textPassword"
              placeholder="可选：设置文本密码保护"
              :maxLength="50"
              allow-clear
            />
            <a-input
              v-model:value="textRemark"
              placeholder="可选：添加备注信息"
              :maxLength="100"
              allow-clear
            />
          </div>
          
          <div class="text-upload-actions">
            <a-button type="primary" @click="uploadText" :loading="textUploading">
              <template #icon><upload-outlined /></template>
              上传文本
            </a-button>
          </div>
        </a-card>

        <!-- 文本列表卡片 -->
        <a-card v-if="textList.length > 0" class="text-list-card" :bordered="false">
          <template #title>
            <span class="card-title">
              <form-outlined /> 已上传文本 ({{ textList.length }})
            </span>
          </template>
          <a-table
            :columns="textColumns"
            :data-source="textList"
            :pagination="{ pageSize: 5 }"
            :row-key="record => record.id"
            size="small"
          >
            <template #bodyCell="{ column, record }">
              <template v-if="column.key === 'content'">
                <div class="text-content">
                  {{ record.content }}
                  <lock-outlined v-if="record.passwordProtected" style="margin-left: 8px; color: #faad14;" />
                </div>
              </template>
              <template v-if="column.key === 'action'">
                <a-space>
                  <a-button type="link" @click="copyText(record)">
                    <template #icon><copy-outlined /></template>
                    复制
                  </a-button>
                  <a-button type="link" danger @click="confirmDeleteText(record)">
                    <template #icon><delete-outlined /></template>
                    删除
                  </a-button>
                </a-space>
              </template>
            </template>
          </a-table>
        </a-card>
      </a-tab-pane>
    </a-tabs>
    
    <!-- 密码验证弹窗 -->
    <a-modal
      v-model:open="passwordModalVisible"
      title="请输入密码"
      @ok="handlePasswordConfirm"
      @cancel="handlePasswordCancel"
      :confirmLoading="passwordVerifying"
    >
      <a-input-password
        v-model:value="inputPassword"
        placeholder="请输入密码"
        @pressEnter="handlePasswordConfirm"
      />
    </a-modal>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import {
  InboxOutlined,
  FolderOutlined,
  FileOutlined,
  DownloadOutlined,
  DeleteOutlined,
  FormOutlined,
  UploadOutlined,
  CopyOutlined,
  LockOutlined
} from '@ant-design/icons-vue'
import { message, Modal } from 'ant-design-vue'
import axios from 'axios'

const activeTab = ref('file')
const fileList = ref([])
const textList = ref([])
const uploadStatus = ref('normal')
const textContent = ref('')
const textUploading = ref(false)
const uploadProgress = ref(0)
const maxFileSize = ref(100) // 默认100MB，从后端动态获取

// 密码相关
const filePassword = ref('')
const fileRemark = ref('')
const textPassword = ref('')
const textRemark = ref('')
const passwordModalVisible = ref(false)
const inputPassword = ref('')
const passwordVerifying = ref(false)
const currentAction = ref(null) // 当前需要密码验证的操作

// 获取文件上传配置
const fetchUploadConfig = async () => {
  try {
    const response = await axios.get('/api/files/config')
    maxFileSize.value = response.data.maxFileSize || 100
  } catch (error) {
    console.error('获取配置失败:', error)
    maxFileSize.value = 100 // 使用默认值
  }
}

// 获取文件列表
const fetchFileList = async () => {
  try {
    const response = await axios.get('/api/files')
    fileList.value = response.data.map((file, index) => ({
      id: file.id,
      fileName: file.fileName,
      fileType: file.fileType,
      fileSize: file.size,
      remark: file.remark || '',
      uploadTime: file.createTime ? new Date(parseInt(file.createTime)).toLocaleString() : new Date().toLocaleString(),
      passwordProtected: file.passwordProtected
    }))
  } catch (error) {
    message.error('获取文件列表失败')
  }
}

// 获取文本列表
const fetchTextList = async () => {
  try {
    const response = await axios.get('/api/texts')
    textList.value = response.data.map(text => ({
      id: text.id,
      content: text.content,
      remark: text.remark || '',
      uploadTime: text.createTime ? new Date(parseInt(text.createTime)).toLocaleString() : new Date().toLocaleString(),
      passwordProtected: text.passwordProtected
    }))
  } catch (error) {
    message.error('获取文本列表失败')
  }
}

// 组件挂载时获取配置和列表
onMounted(() => {
  fetchUploadConfig()
  fetchFileList()
  fetchTextList()
})

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
    title: '备注',
    dataIndex: 'remark',
    key: 'remark',
    width: 150,
    ellipsis: true
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

// 文本列表的列配置
const textColumns = [
  {
    title: '文本内容',
    dataIndex: 'content',
    key: 'content',
    ellipsis: true
  },
  {
    title: '备注',
    dataIndex: 'remark',
    key: 'remark',
    width: 150,
    ellipsis: true
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

const handleUploadSuccess = (response) => {
  uploadStatus.value = 'success'
  message.success('文件上传成功')
  // 重新获取文件列表以获取正确的文件ID
  fetchFileList()
  // 清空密码和备注
  filePassword.value = ''
  fileRemark.value = ''
  setTimeout(() => {
    uploadStatus.value = 'normal'
  }, 1000)
}

const handleUploadError = (error) => {
  uploadStatus.value = 'error'
  
  // 根据后端错误类型显示不同的错误信息
  if (error?.response?.data?.error === 'MAX_SIZE_EXCEEDED') {
    message.error('文件大小超过系统限制')
  } else if (error?.response?.data?.error === 'RUNTIME_ERROR') {
    message.error(error.response.data.message || '文件大小超过限制')
  } else if (error?.response?.data?.message) {
    message.error(error.response.data.message)
  } else {
    message.error('文件上传失败，请重试')
  }
}

const beforeUpload = (file) => {
  const maxSizeBytes = maxFileSize.value * 1024 * 1024
  if (file.size > maxSizeBytes) {
    message.error(`文件大小不能超过${maxFileSize.value}MB`)
    return false
  }
  return true
}

const customRequest = async ({ file, onSuccess, onError }) => {
  const formData = new FormData()
  formData.append('file', file)
  if (filePassword.value) {
    formData.append('password', filePassword.value)
  }
  if (fileRemark.value) {
    formData.append('remark', fileRemark.value)
  }
  uploadProgress.value = 0

  try {
    const response = await axios.post('/api/files/upload', formData, {
      onUploadProgress: (progressEvent) => {
        if (progressEvent.total) {
          uploadProgress.value = Math.round((progressEvent.loaded * 100) / progressEvent.total)
        }
      }
    })
    onSuccess()
    handleUploadSuccess(response.data)
    setTimeout(() => {
      uploadProgress.value = 0
    }, 1000)
  } catch (error) {
    onError()
    handleUploadError(error)
    uploadProgress.value = 0
  }
}

const formatFileSize = (bytes) => {
  if (bytes === 0) return '0 B'
  const k = 1024
  const sizes = ['B', 'KB', 'MB', 'GB', 'TB']
  const i = Math.floor(Math.log(bytes) / Math.log(k))
  return (bytes / Math.pow(k, i)).toFixed(2) + ' ' + sizes[i]
}

// 下载文件
const downloadFile = async (file) => {
  if (file.passwordProtected) {
    // 需要密码验证
    currentAction.value = { type: 'download', data: file }
    passwordModalVisible.value = true
  } else {
    // 直接下载
    await performDownload(file)
  }
}

// 执行下载
const performDownload = async (file, password = '') => {
  try {
    const params = new URLSearchParams()
    if (password) {
      params.append('password', password)
    }
    const url = `/api/files/${file.id}${params.toString() ? '?' + params.toString() : ''}`
    
    const response = await axios({
      url,
      method: 'GET',
      responseType: 'blob'
    })
    
    const downloadUrl = window.URL.createObjectURL(new Blob([response.data]))
    const link = document.createElement('a')
    link.href = downloadUrl
    link.setAttribute('download', file.fileName)
    document.body.appendChild(link)
    link.click()
    document.body.removeChild(link)
    window.URL.revokeObjectURL(downloadUrl)
    message.success('文件下载成功')
  } catch (error) {
    if (error.response?.status === 401) {
      message.error('密码错误')
    } else {
      message.error('文件下载失败')
    }
    throw error
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

const uploadText = async () => {
  if (!textContent.value.trim()) {
    message.warning('请输入文本内容')
    return
  }

  textUploading.value = true
  try {
    const formData = new FormData()
    formData.append('content', textContent.value)
    if (textPassword.value) {
      formData.append('password', textPassword.value)
    }
    if (textRemark.value) {
      formData.append('remark', textRemark.value)
    }
    
    const response = await axios.post('/api/texts/upload', formData)
    
    // 重新获取文本列表
    await fetchTextList()
    
    textContent.value = ''
    textPassword.value = ''
    textRemark.value = ''
    message.success('文本上传成功')
  } catch (error) {
    message.error('文本上传失败')
  } finally {
    textUploading.value = false
  }
}

// 复制文本
const copyText = async (text) => {
  if (text.passwordProtected) {
    // 需要密码验证
    currentAction.value = { type: 'copy', data: text }
    passwordModalVisible.value = true
  } else {
    // 直接复制
    await performCopy(text)
  }
}

// 执行复制
const performCopy = async (text, password = '') => {
  try {
    let content = text.content
    if (text.passwordProtected) {
      // 获取完整内容
      const params = new URLSearchParams()
      if (password) {
        params.append('password', password)
      }
      const url = `/api/texts/${text.id}${params.toString() ? '?' + params.toString() : ''}`
      const response = await axios.get(url)
      content = response.data.content
    }
    await navigator.clipboard.writeText(content)
    message.success('文本已复制到剪贴板')
  } catch (error) {
    if (error.response?.status === 401) {
      message.error('密码错误')
    } else {
      message.error('复制失败')
    }
    throw error
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

// 密码验证相关
const handlePasswordConfirm = async () => {
  if (!inputPassword.value) {
    message.warning('请输入密码')
    return
  }
  
  passwordVerifying.value = true
  try {
    if (currentAction.value.type === 'download') {
      await performDownload(currentAction.value.data, inputPassword.value)
    } else if (currentAction.value.type === 'copy') {
      await performCopy(currentAction.value.data, inputPassword.value)
    }
    handlePasswordCancel()
  } catch (error) {
    // 错误已在具体方法中处理
  } finally {
    passwordVerifying.value = false
  }
}

const handlePasswordCancel = () => {
  passwordModalVisible.value = false
  inputPassword.value = ''
  currentAction.value = null
  passwordVerifying.value = false
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

.upload-options,
.text-upload-options {
  margin-top: 16px;
  display: flex;
  gap: 12px;
}

.upload-options > *,
.text-upload-options > * {
  flex: 1;
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
  display: flex;
  align-items: center;
}

.upload-progress {
  margin-top: 16px;
}
</style>

