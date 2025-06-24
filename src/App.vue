<script setup>
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
  <div class="page-background">
    <main class="main-container">
      <header class="app-header">
        <h1 class="title">
          <svg xmlns="http://www.w3.org/2000/svg" width="36" height="36" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-pen-tool"><path d="M12 19l7-7 3 3-7 7-3-3z"></path><path d="M18 13l-1.5-7.5L2 2l3.5 14.5L13 18l5-5z"></path><path d="M2 2l7.586 7.586"></path><path d="M11 11L2 22"></path></svg>
          周报生成器 Pro
        </h1>
        <p class="subtitle">高效工作，优雅汇报。让AI成为你的职场助理。</p>
      </header>
      
      <div class="input-section">
        <textarea
          v-model="userInput"
          rows="12"
          placeholder="在这里输入本周的工作流水账，越详细越好...
例如：
 - 周一：与产品经理开会，敲定了V2.1版本需求；
 - 周二：修复了用户反馈的3个UI错位bug；
 - 周三：完成了新功能“数据导出”的后端接口开发；
 - 周四：撰写技术文档，并为团队分享了新的代码规范；
 - 周五：协助测试同事排查问题，并规划下周工作。"
        ></textarea>
        
        <button @click="generateReport" :disabled="loading" class="generate-button">
          <span v-if="!loading">✨ 一键生成专业周报</span>
          <span v-else class="loading-animation">
            <svg class="spinner" viewBox="0 0 50 50"><circle class="path" cx="25" cy="25" r="20" fill="none" stroke-width="5"></circle></svg>
            思考中，请稍候...
          </span>
        </button>
      </div>

      <transition name="fade">
        <div v-if="reportResult" class="result-section">
          <h2 class="result-title">AI为您生成的周报</h2>
          <pre class="result-text">{{ reportResult }}</pre>
        </div>
      </transition>
    </main>
    
    <footer class="app-footer">
      <p>Made with ❤️ by WangTong07. 
        <a href="https://github.com/WangTong07/gong-zai-weekly" target="_blank" rel="noopener noreferrer">View on GitHub</a>
      </p>
    </footer>
  </div>
</template>

<style>
/* --- 全局样式与字体 --- */
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@400;500;700&display=swap');

:root {
  --primary-color: #4f46e5;
  --primary-hover-color: #4338ca;
  --background-color: #f8fafc;
  --card-background-color: #ffffff;
  --text-color-primary: #1e293b;
  --text-color-secondary: #64748b;
  --border-color: #e2e8f0;
  --shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
}

body {
  margin: 0;
  font-family: 'Noto Sans SC', sans-serif;
  background-color: var(--background-color);
  color: var(--text-color-primary);
}

/* --- 页面布局 --- */
.page-background {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 100vh;
  padding: 2rem 1rem;
  box-sizing: border-box;
}

.main-container {
  width: 100%;
  max-width: 800px;
}

/* --- 头部 --- */
.app-header {
  text-align: center;
  margin-bottom: 2.5rem;
}
.title {
  font-size: 2.5rem;
  font-weight: 700;
  color: var(--text-color-primary);
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  margin: 0;
}
.title svg {
  color: var(--primary-color);
}
.subtitle {
  font-size: 1.125rem;
  color: var(--text-color-secondary);
  margin-top: 0.5rem;
}

/* --- 输入区域 --- */
.input-section {
  background: var(--card-background-color);
  padding: 2rem;
  border-radius: 0.75rem;
  box-shadow: var(--shadow);
  border: 1px solid var(--border-color);
}

textarea {
  width: 100%;
  padding: 1rem;
  border: 1px solid var(--border-color);
  border-radius: 0.5rem;
  font-size: 1rem;
  line-height: 1.6;
  box-sizing: border-box;
  resize: vertical;
  transition: border-color 0.2s, box-shadow 0.2s;
}
textarea:focus {
  outline: none;
  border-color: var(--primary-color);
  box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.2);
}

/* --- 生成按钮 --- */
.generate-button {
  width: 100%;
  padding: 1rem 1.5rem;
  margin-top: 1.5rem;
  background-color: var(--primary-color);
  color: white;
  font-weight: 500;
  font-size: 1.125rem;
  border: none;
  border-radius: 0.5rem;
  cursor: pointer;
  transition: background-color 0.2s, transform 0.1s;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}
.generate-button:hover:not(:disabled) {
  background-color: var(--primary-hover-color);
}
.generate-button:active:not(:disabled) {
  transform: scale(0.98);
}
.generate-button:disabled {
  background-color: #9ca3af;
  cursor: not-allowed;
}

/* --- 加载动画 --- */
.loading-animation {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}
.spinner {
  animation: rotate 2s linear infinite;
  width: 24px;
  height: 24px;
}
.path {
  stroke: rgba(255, 255, 255, 0.8);
  stroke-linecap: round;
  animation: dash 1.5s ease-in-out infinite;
}
@keyframes rotate { 100% { transform: rotate(360deg); } }
@keyframes dash {
  0% { stroke-dasharray: 1, 150; stroke-dashoffset: 0; }
  50% { stroke-dasharray: 90, 150; stroke-dashoffset: -35; }
  100% { stroke-dasharray: 90, 150; stroke-dashoffset: -124; }
}

/* --- 结果区域 --- */
.result-section {
  margin-top: 2.5rem;
  background: var(--card-background-color);
  padding: 2rem;
  border-radius: 0.75rem;
  box-shadow: var(--shadow);
  border: 1px solid var(--border-color);
}
.result-title {
  font-size: 1.5rem;
  font-weight: 500;
  margin-top: 0;
  margin-bottom: 1.5rem;
  border-bottom: 1px solid var(--border-color);
  padding-bottom: 1rem;
}
.result-text {
  white-space: pre-wrap;
  word-wrap: break-word;
  color: #334155;
  font-size: 1rem;
  line-height: 1.7;
  background-color: #f8fafc;
  padding: 1rem;
  border-radius: 0.5rem;
}

/* --- 过渡动画 --- */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

/* --- 页脚 --- */
.app-footer {
  text-align: center;
  margin-top: 3rem;
  color: var(--text-color-secondary);
  font-size: 0.875rem;
}
.app-footer a {
  color: var(--primary-color);
  text-decoration: none;
  font-weight: 500;
}
.app-footer a:hover {
  text-decoration: underline;
}
</style>