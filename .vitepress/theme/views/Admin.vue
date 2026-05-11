<template>
  <div class="admin-container">
    <!-- 登录界面 -->
    <div v-if="!isLoggedIn" class="login-card s-card">
      <h2>管理后台</h2>
      <div class="input-group">
        <input
          type="password"
          v-model="password"
          placeholder="请输入密码"
          @keyup.enter="login"
        />
      </div>
      <button @click="login">登录</button>
      <p v-if="error" class="error">{{ error }}</p>
    </div>

    <!-- 管理界面 -->
    <div v-else class="admin-panel">
      <div class="header">
        <h2>文章管理助手</h2>
        <button @click="showWritePanel = !showWritePanel" class="btn-write">
          {{ showWritePanel ? '隐藏' : '写文章' }}
        </button>
      </div>

      <!-- 说明 -->
      <div class="notice s-card">
        <h3>使用说明</h3>
        <p>由于浏览器安全限制，此工具生成文章内容后需要手动保存到 <code>page/</code> 目录</p>
        <p>文章会按日期自动创建目录结构</p>
      </div>

      <!-- 写文章面板 -->
      <div v-if="showWritePanel" class="write-panel s-card">
        <div class="form-group">
          <label>标题</label>
          <input v-model="newArticle.title" placeholder="文章标题" />
        </div>
        <div class="form-group">
          <label>日期</label>
          <input v-model="newArticle.date" type="date" />
        </div>
        <div class="form-group">
          <label>分类</label>
          <input v-model="newArticle.category" placeholder="文章分类" />
        </div>
        <div class="form-group">
          <label>标签 (逗号分隔)</label>
          <input v-model="newArticle.tagsInput" placeholder="标签1,标签2,标签3" />
        </div>
        <div class="form-group">
          <label>封面图片 (可选)</label>
          <input v-model="newArticle.cover" placeholder="封面图片路径" />
        </div>
        <div class="form-group">
          <label>内容</label>
          <textarea v-model="newArticle.content" placeholder="文章内容"></textarea>
        </div>
        <div class="form-actions">
          <button @click="generateArticle">生成文章</button>
          <button @click="resetForm" class="btn-secondary">重置</button>
        </div>
      </div>

      <!-- 预览和复制 -->
      <div v-if="generatedContent" class="preview-panel s-card">
        <h3>生成的文章</h3>
        <div class="file-info">
          <p><strong>保存位置:</strong> <code>{{ filePath }}</code></p>
        </div>
        <textarea readonly class="preview-area">{{ generatedContent }}</textarea>
        <button @click="copyToClipboard" class="btn-copy">复制内容</button>
        <p v-if="copied" class="copy-success">已复制到剪贴板！</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const isLoggedIn = ref(false)
const password = ref('')
const error = ref('')
const showWritePanel = ref(true)
const generatedContent = ref('')
const copied = ref(false)

const newArticle = ref({
  title: '',
  date: new Date().toISOString().split('T')[0],
  category: '',
  tagsInput: '',
  cover: '',
  content: ''
})

const filePath = computed(() => {
  const dateObj = new Date(newArticle.value.date)
  const year = dateObj.getFullYear()
  const month = String(dateObj.getMonth() + 1).padStart(2, '0')
  const day = String(dateObj.getDate()).padStart(2, '0')
  return `page/${year}-${month}-${day}/${Date.now()}.md`
})

const ADMIN_PASSWORD = 'gzx140715'

const login = () => {
  if (password.value === ADMIN_PASSWORD) {
    isLoggedIn.value = true
    error.value = ''
    localStorage.setItem('adminLoggedIn', 'true')
  } else {
    error.value = '密码错误'
  }
}

const generateArticle = () => {
  const tagsArray = newArticle.value.tagsInput.split(',').map(tag => tag.trim()).filter(tag => tag)
  
  let content = '---\n'
  content += `title: "${newArticle.value.title}"\n`
  content += `date: "${newArticle.value.date}"\n`
  if (newArticle.value.category) {
    content += `category: "${newArticle.value.category}"\n`
  }
  if (tagsArray.length > 0) {
    content += `tags:\n`
    tagsArray.forEach(tag => {
      content += `  - "${tag}"\n`
    })
  }
  if (newArticle.value.cover) {
    content += `cover: "${newArticle.value.cover}"\n`
  }
  content += `articleGPT: true\n`
  content += '---\n\n'
  content += newArticle.value.content
  
  generatedContent.value = content
}

const copyToClipboard = () => {
  navigator.clipboard.writeText(generatedContent.value).then(() => {
    copied.value = true
    setTimeout(() => {
      copied.value = false
    }, 2000)
  })
}

const resetForm = () => {
  newArticle.value = {
    title: '',
    date: new Date().toISOString().split('T')[0],
    category: '',
    tagsInput: '',
    cover: '',
    content: ''
  }
  generatedContent.value = ''
}

import { onMounted } from 'vue'
onMounted(() => {
  const stored = localStorage.getItem('adminLoggedIn')
  if (stored === 'true') {
    isLoggedIn.value = true
  }
})
</script>

<style lang="scss" scoped>
.admin-container {
  max-width: 900px;
  margin: 0 auto;
  padding: 2rem;
}

.login-card {
  max-width: 400px;
  margin: 0 auto;
  padding: 2rem;
  text-align: center;
}

.login-card h2 {
  margin-bottom: 1.5rem;
}

.input-group {
  margin-bottom: 1rem;
}

.input-group input {
  width: 100%;
  padding: 0.8rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
}

button {
  padding: 0.8rem 1.5rem;
  background: var(--main-color);
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
}

button:hover {
  opacity: 0.9;
}

.error {
  color: #ff4444;
  margin-top: 1rem;
}

.admin-panel {
  h2 {
    margin-bottom: 1.5rem;
  }
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
}

.btn-write {
  background: #4CAF50;
}

.notice {
  padding: 1.5rem;
  margin-bottom: 1.5rem;
  background: #fff3cd;
  border-left: 4px solid #ffc107;
  
  h3 {
    margin: 0 0 0.5rem 0;
    color: #856404;
  }
  
  p {
    margin: 0.3rem 0;
    color: #856404;
  }
  
  code {
    background: rgba(0,0,0,0.1);
    padding: 0.2rem 0.4rem;
    border-radius: 3px;
  }
}

.write-panel {
  padding: 1.5rem;
  margin-bottom: 1.5rem;
}

.form-group {
  margin-bottom: 1rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: bold;
}

.form-group input,
.form-group textarea {
  width: 100%;
  padding: 0.8rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
}

.form-group textarea {
  min-height: 300px;
  resize: vertical;
}

.form-actions {
  display: flex;
  gap: 1rem;
  margin-top: 1.5rem;
}

.btn-secondary {
  background: #666;
}

.preview-panel {
  padding: 1.5rem;
  
  h3 {
    margin: 0 0 1rem 0;
  }
}

.file-info {
  background: #f5f5f5;
  padding: 1rem;
  border-radius: 4px;
  margin-bottom: 1rem;
  
  p {
    margin: 0;
  }
  
  code {
    background: #e0e0e0;
    padding: 0.2rem 0.4rem;
    border-radius: 3px;
  }
}

.preview-area {
  width: 100%;
  min-height: 400px;
  padding: 1rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-family: monospace;
  font-size: 0.9rem;
  margin-bottom: 1rem;
}

.btn-copy {
  background: #2196F3;
}

.copy-success {
  color: #4CAF50;
  margin-top: 0.5rem;
}
</style>
