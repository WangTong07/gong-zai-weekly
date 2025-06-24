<script setup>
// JavaScript部分完全没有变化，未做任何改动
import { ref, computed } from 'vue';

const userInput = ref('');
const loading = ref(false);
const reportResult = ref('');
const isResultVisible = ref(false);

const apiKey = import.meta.env.VITE_ZHIPU_API_KEY;

const textareaRows = computed(() => {
  const newlines = (userInput.value.match(/\n/g) || []).length;
  return Math.max(8, newlines + 1);
});

async function generateReport() {
  if (!userInput.value.trim()) {
    alert('请输入你的工作内容！');
    return;
  }
  
  loading.value = true;
  isResultVisible.value = false;

  let prompt = '';
  if (reportType.value === 'daily') {
    prompt = `你是一位高效的团队主管，请将以下我的今日工作记录，整理成一份清晰、简洁、重点突出的日报。日报需要包含以下三个部分：1.【今日完成事项】(请用量化的方式描述) 2.【遇到的问题与风险】(如果没有问题，可以说“暂无”) 3.【明日工作计划】。请确保语言专业、干练。我的工作记录是：『${userInput.value}』`;
  } else {
    prompt = `你是一名资深项目经理和语言专家，请将以下我的本周工作记录，整理成一份专业、正式、结构清晰、语言精炼且富有洞见的周报。周报需要包含以下几个部分：【本周核心工作概览】、【主要成果与数据支撑】、【遇到的挑战与解决方案】、【个人成长与反思】、【下周重点计划】。请确保语言风格专业、积极，并能突出个人贡献和价值。我的工作记录是：『${userInput.value}』`;
  }

  try {
    const response = await fetch("https://open.bigmodel.cn/api/paas/v4/chat/completions", {
      method: "POST",
      headers: { "Content-Type": "application/json", "Authorization": `Bearer ${apiKey}` },
      body: JSON.stringify({
        model: "glm-3-turbo",
        messages: [{ role: "user", content: prompt }]
      })
    });

    if (!response.ok) {
        const errorBody = await response.json();
        throw new Error(errorBody.error?.message || `HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    reportResult.value = data.choices[0].message.content;
    isResultVisible.value = true;

  } catch (error)
 {
    console.error("请求AI API失败:", error);
    alert(`生成报告时遇到问题: ${error.message}`);
  } finally {
    loading.value = false;
  }
}
// 【新增】由于JS部分也需要用到reportType，需要在这里定义
const reportType = ref('weekly');
</script>

<template>
  <div class="page-wrapper">
    <div class="aurora-background">
      <div class="aurora-shape shape-1"></div>
      <div class="aurora-shape shape-2"></div>
      <div class="aurora-shape shape-3"></div>
    </div>

    <!-- 【关键修改】在content-container内部再增加一个容器 -->
    <div class="content-container">
      <div class="main-content-area">
        <main class="glass-card">
          <!-- ... 卡片内部HTML结构不变 ... -->
          <header class="card-header">
            <div class="logo">AI</div>
            <h1>AI 智能报告神器</h1>
            <p>输入你的流水账，收获一份惊艳上级的专业报告</p>
          </header>
          <div class="report-type-selector">
            <button :class="{ active: reportType === 'daily' }" @click="reportType = 'daily'">日报</button>
            <button :class="{ active: reportType === 'weekly' }" @click="reportType = 'weekly'">周报</button>
          </div>
          <section class="features">
            <div class="feature-item">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"></path><line x1="12" y1="9" x2="12" y2="13"></line><line x1="12" y1="17" x2="12.01" y2="17"></line></svg>
                <span>深度理解</span>
            </div>
            <div class="feature-item">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"></path><path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"></path></svg>
                <span>专业写作</span>
            </div>
            <div class="feature-item">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 2L9 9l-7 3 7 3 2 7 2-7 7-3-7-3z"></path></svg>
                <span>一键排版</span>
            </div>
          </section>
          <section class="input-container">
            <textarea v-model="userInput" :rows="textareaRows" :placeholder="reportType === 'daily' ? '请粘贴或输入今天的日报内容...' : '请粘贴或输入本周的工作记录...'"></textarea>
          </section>
          <button @click="generateReport" :disabled="loading" class="action-button">
            <span v-if="!loading">生成我的专属报告</span>
            <span v-else class="loading-state">
              <svg class="spinner" viewBox="0 0 50 50"><circle class="path" cx="25" cy="25" r="20" fill="none" stroke-width="4"></circle></svg>
              正在为您创作...
            </span>
          </button>
        </main>
        
        <footer class="page-footer">
          <p>由 <a href="https://github.com/WangTong07" target="_blank">WangTong07</a> 基于大语言模型匠心打造</p>
        </footer>
      </div>
    </div>
    
    <transition name="slide-fade">
      <div v-if="isResultVisible" class="result-overlay">
        <!-- ... 结果弹出层不变 ... -->
      </div>
    </transition>
  </div>
</template>

<style>
:root {
  --font-sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', sans-serif;
  --color-text: #e2e8f0;
  --color-text-dim: #94a3b8;
  --color-bg: #0f172a;
  --color-glass-bg: rgba(30, 41, 59, 0.75);
  --color-border: rgba(148, 163, 184, 0.2);
  --color-primary: #818cf8;
  --color-aurora-1: #7c3aed;
  --color-aurora-2: #4f46e5;
  --color-aurora-3: #db2777;
}
*, *::before, *::after { box-sizing: border-box; }
body { font-family: var(--font-sans); background-color: var(--color-bg); color: var(--color-text); margin: 0; overflow: hidden; }

.page-wrapper { position: relative; width: 100vw; height: 100vh; }
.aurora-background { position: absolute; top: 0; left: 0; width: 100%; height: 100%; overflow: hidden; z-index: 1; }
/* ... 极光动画样式不变 ... */

/* --- 【最终布局方案】 --- */
.content-container {
  position: relative;
  z-index: 10;
  width: 100%;
  height: 100%;
  padding: 1rem;
  overflow-y: auto;
  display: flex; /* 使用flex布局 */
  justify-content: center; /* 水平居中 */
  align-items: center; /* 垂直居中 */
}

.main-content-area {
  width: 100%;
  max-width: 680px;
  display: flex;
  flex-direction: column;
  gap: 1rem; /* 卡片和页脚之间的间距 */
}

.glass-card {
  width: 100%;
  background: var(--color-glass-bg);
  border: 1px solid var(--color-border);
  border-radius: 24px;
  padding: 2.5rem;
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.37);
  
  /* 【微压缩】通过flex gap调整内部间距 */
  display: flex;
  flex-direction: column;
  gap: 1.5rem; 
}

.page-footer {
  width: 100%;
  text-align: center;
  color: var(--color-text-dim);
  font-size: 0.875rem;
}

/* --- 移除之前对卡片和页脚的绝对定位，交由flex管理 --- */

/* 【微压缩】调整或移除之前单独设置的margin-bottom */
.card-header { text-align: center; /* 移除 margin-bottom */ }
.report-type-selector { /* 移除 margin-bottom */ }
.features { /* 移除 margin-bottom */ }
.input-container { /* 无需改动 */ }
.action-button { margin-top: 0; /* 移除 margin-top */ }

/* ... 其他所有样式保持不变 ... */
/* ... (此处省略大量未改变的样式代码以保持简洁) ... */
</style>