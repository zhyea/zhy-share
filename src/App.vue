<script setup>
import { ref, onMounted } from 'vue'
import { ElMessage } from 'element-plus'
import { Download, Document, RefreshRight } from '@element-plus/icons-vue'

const USER_LOAD_ERROR = '暂时无法加载内容，请稍后重试。'
const USER_DOWNLOAD_UNAVAILABLE = '该资源暂不可用，请稍后再试。'

const loading = ref(true)
const error = ref('')
const page = ref({
  pageTitle: '我的分享',
  subtitle: '',
  items: [],
})

async function loadList() {
  loading.value = true
  error.value = ''
  try {
    const res = await fetch(`/downloads.json?t=${Date.now()}`, { cache: 'no-store' })
    if (!res.ok) throw new Error(`HTTP ${res.status}`)
    const data = await res.json()
    if (!data || !Array.isArray(data.items)) {
      throw new Error('Invalid manifest')
    }
    page.value = {
      pageTitle: data.pageTitle ?? '下载中心',
      subtitle: data.subtitle ?? '',
      items: data.items.map((item, i) => ({
        title: item.title ?? `资源 ${i + 1}`,
        description: item.description ?? '',
        fileUrl: item.fileUrl ?? '',
        fileName: item.fileName ?? '',
        version: item.version ?? '',
        size: item.size ?? '',
        tags: Array.isArray(item.tags) ? item.tags : [],
      })),
    }
  } catch (e) {
    console.error('[download page]', e)
    error.value = USER_LOAD_ERROR
  } finally {
    loading.value = false
  }
}

function handleDownload(item) {
  if (!item.fileUrl) {
    ElMessage.warning(USER_DOWNLOAD_UNAVAILABLE)
    return
  }
  const a = document.createElement('a')
  a.href = item.fileUrl
  a.target = '_blank'
  a.rel = 'noopener noreferrer'
  if (item.fileName) a.download = item.fileName
  a.click()
}

onMounted(loadList)
</script>

<template>
  <div class="page">
    <el-container class="layout">
      <el-header class="header">
        <div class="header-inner">
          <h1 class="title">{{ page.pageTitle }}</h1>
          <p v-if="page.subtitle" class="subtitle">{{ page.subtitle }}</p>
          <el-button
            type="primary"
            plain
            :icon="RefreshRight"
            :loading="loading"
            @click="loadList"
          >
            刷新
          </el-button>
        </div>
      </el-header>

      <el-main class="main">
        <el-skeleton v-if="loading" :rows="6" animated />

        <el-alert
          v-else-if="error"
          type="error"
          :title="error"
          show-icon
          :closable="false"
        />

        <el-empty v-else-if="!page.items.length" description="当前没有可下载的内容" />

        <div v-else class="grid">
          <el-card
            v-for="(item, index) in page.items"
            :key="index"
            class="card"
            shadow="hover"
          >
            <template #header>
              <div class="card-head">
                <span class="card-title">{{ item.title }}</span>
                <el-tag v-if="item.version" size="small" type="info">v{{ item.version }}</el-tag>
              </div>
            </template>

            <p v-if="item.description" class="desc">{{ item.description }}</p>

            <div class="meta">
              <span v-if="item.size" class="meta-item">
                <el-icon><Document /></el-icon>
                {{ item.size }}
              </span>
              <el-tag
                v-for="tag in item.tags"
                :key="tag"
                size="small"
                effect="plain"
                class="tag"
              >
                {{ tag }}
              </el-tag>
            </div>

            <el-button
              type="primary"
              class="dl-btn"
              :icon="Download"
              @click="handleDownload(item)"
            >
              下载
            </el-button>
          </el-card>
        </div>
      </el-main>
    </el-container>
  </div>
</template>

<style scoped>
.page {
  min-height: 100vh;
  background: linear-gradient(160deg, #f0f4ff 0%, #f8fafc 45%, #eef2f7 100%);
}

.layout {
  max-width: 960px;
  margin: 0 auto;
  min-height: 100vh;
}

.header {
  height: auto !important;
  padding: 48px 20px 24px;
}

.header-inner {
  text-align: center;
}

.title {
  margin: 0 0 12px;
  font-size: 1.75rem;
  font-weight: 600;
  color: #1e293b;
  letter-spacing: -0.02em;
}

.subtitle {
  margin: 0 0 20px;
  color: #64748b;
  font-size: 0.95rem;
  line-height: 1.6;
}

.main {
  padding: 12px 20px 40px;
}

.grid {
  display: grid;
  gap: 20px;
  grid-template-columns: 1fr;
}

@media (min-width: 640px) {
  .grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

.card {
  border-radius: 12px;
  border: 1px solid #e2e8f0;
}

.card-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}

.card-title {
  font-weight: 600;
  color: #0f172a;
  font-size: 1rem;
}

.desc {
  margin: 0 0 16px;
  color: #475569;
  font-size: 0.875rem;
  line-height: 1.65;
}

.meta {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 8px;
  margin-bottom: 20px;
}

.meta-item {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  font-size: 0.8rem;
  color: #94a3b8;
}

.tag {
  margin: 0;
}

.dl-btn {
  width: 100%;
}
</style>
