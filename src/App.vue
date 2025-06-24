<script setup>
// JavaScript部分完全没有变化，所以这里省略了
// 我们只关注 <template> 和 <style> 的最终形态
import { ref } from 'vue';

const userInput = ref('');
const loading = ref(false);
const reportResult = ref('');

const apiKey = import.meta.env.VITE_ZHIPU_API_KEY;

async function generateReport() {
  if (!userInput.value.trim()) {
    alert('请输入你的工作内容！');
    return;
  }
  
  loading.value = true;
  reportResult.value = '';

  const prompt = `你是一名资深项目经理，请将以下我的本周工作记录，整理成一份专业、正式、向上汇报的周报。请分点阐述，并适当润色，突出工作亮点和价值。周报需要包含以下几个部分：1. 本周重点工作总结 2. 主要成果与数据 3. 下周工作计划。我的工作记录是：『${userInput.value}』`;

  try {
    const response = await fetch("https://open.bigmodel.cn/api/paas/v4/chat/completions", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${apiKey}`
      },
      body: JSON.stringify({
        model: "glm-3-turbo",
        messages: [{ role: "user", content: prompt }]
      })
    });

    if (!response.ok) {
        const errorBody = await response.json();
        console.error("API 错误响应:", errorBody);
        alert(`请求失败: ${errorBody.error?.message || response.statusText}`);
        throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    reportResult.value = data.choices[0].message.content;

  } catch (error) {
    console.error("请求AI API失败:", error);
    if (!alert.toString().includes('请求失败')) {
        alert('生成报告时遇到问题，请稍后再试或查看控制台获取详情。');
    }
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <div class="page-container">
    <main class="main-card">
      
      <header class="card-header">
        <div class="header-icon-wrapper">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 20V10M18 20V4M6 20V16"/></svg>
        </div>
        <h1>周报生成器 Pro</h1>
        <p>高效工作，优雅汇报。让AI成为你的职场助理。</p>
      </header>

      <div class="features-grid">
        <div class="feature-card">
          <div class="feature-icon">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2z"></polygon></svg>
          </div>
          <h3>智能生成</h3>
          <p>AI自动整理工作内容</p>
        </div>
        <div class="feature-card">
          <div class="feature-icon">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path><polyline points="14 2 14 8 20 8"></polyline><line x1="16" y1="13" x2="8" y2="13"></line><line x1="16" y1="17" x2="8" y2="17"></line><polyline points="10 9 9 9 8 9"></polyline></svg>
          </div>
          <h3>专业格式</h3>
          <p>规范化周报模板</p>
        </div>
        <div class="feature-card">
          <div class="feature-icon">
             <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 2.69l5.66 5.66a8 8 0 1 1-11.31 0z"></path></svg>
          </div>
          <h3>一键美化</h3>
          <p>专业排版，即用即得</p>
        </div>
      </div>

      <div class="input-area">
        <label for="work-content">请输入本周的工作内容:</label>
        <textarea
          id="work-content"
          v-model="userInput"
          rows="10"
          placeholder="在这里输入本周的工作流水账，越详细越好...
例如：
 - 周一：与产品经理开会，敲定了V2.1版本需求；
 - 周二：修复了用户反馈的3个UI错位bug；
 - 周三：完成了新功能“数据导出”的后端接口开发；"
        ></textarea>
      </div>

      <button @click="generateReport" :disabled="loading" class="submit-button">
        <span v-if="!loading" class="button-content">
          <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 2L9 9l-7 3 7 3 2 7 2-7 7-3-7-3z"></path></svg>
          一键生成专业周报
        </span>
        <span v-else class="loading-content">
          <svg class="spinner" viewBox="0 0 50 50"><circle class="path" cx="25" cy="25" r="20" fill="none" stroke-width="5"></circle></svg>
          AI思考中...
        </span>
      </button>

      <transition name="fade">
        <div v-if="reportResult" class="result-area">
          <pre>{{ reportResult }}</pre>
        </div>
      </transition>

    </main>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');

:root {
  --primary-color-start: #6d28d9;
  --primary-color-end: #4f46e5;
  --background-gradient: linear-gradient(135deg, #f3e8ff 0%, #e0e7ff 100%);
  --card-bg: #ffffff;
  --text-primary: #111827;
  --text-secondary: #6b7280;
  --feature-card-bg: #f9fafb;
  --border-color: #e5e7eb;
}

body {
  margin: 0;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Helvetica Neue', Arial, sans-serif;
  color: var(--text-primary);
}

.page-container {
  display: flex;
  justify-content: center;
  /* --- 关键修改！从 flex-start 改为 center --- */
  align-items: center; 
  min-height: 100vh;
  background: var(--background-gradient);
  padding: 2rem 1rem; /* 调整了padding，让小屏幕体验更好 */
  box-sizing: border-box;
}

.main-card {
  width: 100%;
  max-width: 720px;
  background: var(--card-bg);
  border-radius: 24px;
  padding: 2.5rem 3rem;
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.15);
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

/* 媒体查询：让小屏幕（如手机）布局更好看 */
@media (max-width: 768px) {
  .page-container {
    padding: 1rem;
    align-items: flex-start; /* 小屏幕时靠上对齐，避免键盘弹出时布局混乱 */
  }
  .main-card {
    padding: 1.5rem;
  }
  .features-grid {
    grid-template-columns: 1fr; /* 小屏幕时，三个特性卡片竖向排列 */
  }
  .card-header h1 {
    font-size: 1.75rem;
  }
  .card-header p {
    font-size: 1rem;
  }
}


.card-header { text-align: center; }
.card-header .header-icon-wrapper {
  display: inline-flex;
  justify-content: center;
  align-items: center;
  width: 56px;
  height: 56px;
  border-radius: 12px;
  background: linear-gradient(135deg, var(--primary-color-start), var(--primary-color-end));
  color: white;
  margin-bottom: 1rem;
}
.card-header h1 {
  font-size: 2.25rem;
  font-weight: 700;
  margin: 0;
}
.card-header p {
  font-size: 1.125rem;
  color: var(--text-secondary);
  margin: 0.5rem 0 0;
}

.features-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
}

.feature-card {
  background: var(--feature-card-bg);
  border: 1px solid var(--border-color);
  border-radius: 16px;
  padding: 1.5rem;
  text-align: center;
}
.feature-card .feature-icon {
  display: inline-flex;
  justify-content: center;
  align-items: center;
  width: 40px;
  height: 40px;
  border-radius: 8px;
  background-color: #eef2ff;
  color: var(--primary-color-end);
  margin-bottom: 1rem;
}
.feature-card h3 {
  font-size: 1rem;
  font-weight: 600;
  margin: 0;
}
.feature-card p {
  font-size: 0.875rem;
  color: var(--text-secondary);
  margin: 0.25rem 0 0;
}

.input-area label {
  display: block;
  font-weight: 600;
  margin-bottom: 0.75rem;
}
.input-area textarea {
  width: 100%;
  padding: 1rem;
  border: 1px solid var(--border-color);
  border-radius: 12px;
  font-size: 1rem;
  line-height: 1.6;
  box-sizing: border-box;
  resize: vertical;
  transition: border-color 0.2s, box-shadow 0.2s;
}
.input-area textarea:focus {
  outline: none;
  border-color: var(--primary-color-end);
  box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.2);
}

.submit-button {
  width: 100%;
  padding: 1rem;
  border: none;
  border-radius: 12px;
  background: linear-gradient(135deg, var(--primary-color-start), var(--primary-color-end));
  color: white;
  font-size: 1.125rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  box-shadow: 0 4px 6px rgba(109, 40, 217, 0.2);
}
.submit-button:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 7px 10px rgba(109, 40, 217, 0.25);
}
.submit-button:disabled {
  background: #9ca3af;
  cursor: not-allowed;
  box-shadow: none;
}
.submit-button .button-content, .submit-button .loading-content {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
}

.result-area {
  margin-top: 1rem;
  background: #f9fafb;
  border: 1px solid var(--border-color);
  border-radius: 12px;
  padding: 1.5rem;
}
.result-area pre {
  white-space: pre-wrap;
  word-wrap: break-word;
  font-family: 'Inter', sans-serif;
  font-size: 1rem;
  line-height: 1.7;
  margin: 0;
}

.spinner { animation: rotate 2s linear infinite; width: 24px; height: 24px; }
.path { stroke: rgba(255, 255, 255, 0.9); stroke-linecap: round; animation: dash 1.5s ease-in-out infinite; }
@keyframes rotate { 100% { transform: rotate(360deg); } }
@keyframes dash {
  0% { stroke-dasharray: 1, 150; stroke-dashoffset: 0; }
  50% { stroke-dasharray: 90, 150; stroke-dashoffset: -35; }
  100% { stroke-dasharray: 90, 150; stroke-dashoffset: -124; }
}

.fade-enter-active, .fade-leave-active { transition: opacity 0.5s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; }
</style>