<script setup>
import FileUpload from './components/FileUpload.vue'
import { CloudUploadOutlined } from '@ant-design/icons-vue'
</script>

<template>
  <a-layout class="app-container">
    <a-layout-header class="app-header">
      <div class="header-content">
        <cloud-upload-outlined class="logo-icon" />
        <h1>文件分享平台</h1>
      </div>
    </a-layout-header>
    <a-layout-content class="app-content">
      <div class="content-wrapper">
        <FileUpload />
      </div>
    </a-layout-content>
    <a-layout-footer class="app-footer">
      File Sharing Platform ©2024 Created by Ant Design Vue
    </a-layout-footer>
  </a-layout>
</template>

<style scoped>
.app-container {
  min-height: 100vh;
}

.app-header {
  background: #fff;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  position: fixed;
  z-index: 1;
}

.header-content {
  height: 100%;
  display: flex;
  align-items: center;
  max-width: 1400px;
  margin: 0 auto;
}

.logo-icon {
  font-size: 24px;
  color: #1890ff;
  margin-right: 12px;
}

.app-header h1 {
  margin: 0;
  font-size: 18px;
  color: rgba(0, 0, 0, 0.85);
  font-weight: 600;
}

.app-content {
  padding-top: 88px;
  min-height: calc(100vh - 70px);
}

.content-wrapper {
  max-width: 1400px;
  margin: 0 auto;
  width: 100%;
}

.app-footer {
  text-align: center;
  color: rgba(0, 0, 0, 0.45);
}
</style>
