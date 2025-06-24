<script setup>
// JavaScript逻辑部分与之前完全相同，未做任何改动
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

  const prompt = `你是一名资深项目经理和语言专家，请将以下我的本周工作记录，整理成一份专业、正式、结构清晰、语言精炼且富有洞见的周报。周报需要包含以下几个部分：【本周核心工作概览】、【主要成果与数据支撑】、【遇到的挑战与解决方案】、【个人成长与反思】、【下周重点计划】。请确保语言风格专业、积极，并能突出个人贡献和价值。我的工作记录是：『${userInput.value}』`;

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
</script>

<template>
  <div class="page-wrapper">
    <div class="aurora-background">
      <div class="aurora-shape shape-1"></div>
      <div class="aurora-shape shape-2"></div>
      <div class="aurora-shape shape-3"></div>
    </div>

    <!-- 主卡片直接放在wrapper里，由CSS绝对定位 -->
    <main class="glass-card">
      
      <header class="card-header">
        <div class="logo">AI</div>
        <h1>AI 智能周报神器</h1>
        <p>输入你的流水账，收获一份惊艳上级的专业报告</p>
      </header>

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
        <textarea
          v-model="userInput"
          :rows="textareaRows"
          placeholder="请粘贴或输入本周的工作记录..."
        ></textarea>
      </section>

      <button @click="generateReport" :disabled="loading" class="action-button">
        <span v-if="!loading">生成我的专属周报</span>
        <span v-else class="loading-state">
          <svg class="spinner" viewBox="0 0 50 50"><circle class="path" cx="25" cy="25" r="20" fill="none" stroke-width="4"></circle></svg>
          正在为您创作...
        </span>
      </button>
    </main>
    
    <footer class="page-footer">
      <p>由 <a href="https://github.com/WangTong07" target="_blank">WangTong07</a> 基于大语言模型匠心打造</p>
    </footer>
    
    <transition name="slide-fade">
      <div v-if="isResultVisible" class="result-overlay">
        <div class="result-card">
          <div class="result-header">
            <h3>生成结果</h3>
            <button @click="isResultVisible=false" class="close-btn">×</button>
          </div>
          <div class="result-content">
            <pre>{{ reportResult }}</pre>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<style>
/* --- CSS变量与全局设置 --- */
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
body {
  font-family: var(--font-sans);
  background-color: var(--color-bg);
  color: var(--color-text);
  margin: 0;
  overflow: hidden; /* 彻底禁止页面滚动 */
}

/* --- 【磐石之心 V2 布局】 --- */
.page-wrapper {
  position: relative;
  width: 100vw;
  height: 100vh;
}

.aurora-background {
  position: absolute; top: 0; left: 0;
  width: 100%; height: 100%;
  overflow: hidden; z-index: 1;
}

.glass-card {
  /* 绝对定位 */
  position: absolute;
  /* 移到中心点 */
  top: 50%;
  left: 50%;
  /* 根据自身尺寸回移，实现完美居中 */
  transform: translate(-50%, -50%);

  /* 自身样式 */
  width: calc(100% - 2rem);
  max-width: 680px;
  background: var(--color-glass-bg);
  border: 1px solid var(--color-border);
  border-radius: 24px;
  padding: 2.5rem;
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.37);
  z-index: 10;
}

.page-footer {
  position: absolute;
  bottom: 1rem;
  left: 50%;
  transform: translateX(-50%);
  width: 100%;
  text-align: center;
  color: var(--color-text-dim);
  font-size: 0.875rem;
  z-index: 11;
}

/* ... 其他所有样式保持不变 ... */
.aurora-shape { position: absolute; filter: blur(120px); opacity: 0.3; border-radius: 50%; animation: move 20s infinite alternate; }
.shape-1 { width: 500px; height: 500px; background-color: var(--color-aurora-1); top: -20%; left: -20%; }
.shape-2 { width: 400px; height: 400px; background-color: var(--color-aurora-2); top: 30%; right: -10%; animation-delay: 5s; }
.shape-3 { width: 450px; height: 450px; background-color: var(--color-aurora-3); bottom: -25%; left: 10%; animation-delay: 10s; }
@keyframes move { from { transform: translate(0, 0) rotate(0deg); } to { transform: translate(100px, 50px) rotate(60deg); } }
.result-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background-color: rgba(15, 23, 42, 0.5); backdrop-filter: blur(8px); -webkit-backdrop-filter: blur(8px); display: flex; justify-content: center; align-items: center; z-index: 100; }
.result-card { width: 90%; max-width: 800px; height: 80vh; background: var(--color-glass-bg); border: 1px solid var(--color-border); border-radius: 24px; padding: 2rem; box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.37); display: flex; flex-direction: column; }
.result-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 1rem; flex-shrink: 0; }
.result-header h3 { margin: 0; font-size: 1.5rem; }
.close-btn { background: none; border: none; font-size: 2.5rem; color: var(--color-text-dim); cursor: pointer; transition: color 0.3s ease; line-height: 1; padding: 0; }
.close-btn:hover { color: var(--color-text); }
.result-content { flex-grow: 1; overflow-y: auto; background: rgba(15, 23, 42, 0.7); border-radius: 8px; padding: 1.5rem; }
.result-content pre { white-space: pre-wrap; word-wrap: break-word; font-family: var(--font-sans); font-size: 1rem; line-height: 1.8; margin: 0; color: #cbd5e1; }
.slide-fade-enter-active, .slide-fade-leave-active { transition: all 0.5s cubic-bezier(0.16, 1, 0.3, 1); }
.slide-fade-enter-from, .slide-fade-leave-to { opacity: 0; transform: scale(0.95) translateY(20px); }
.card-header { text-align: center; margin-bottom: 1.5rem; }
.logo { display: inline-block; background: linear-gradient(135deg, var(--color-aurora-1), var(--color-aurora-2)); color: white; width: 48px; height: 48px; border-radius: 12px; font-weight: 700; font-size: 1.5rem; line-height: 48px; margin-bottom: 1rem; }
.card-header h1 { font-size: 2rem; margin: 0; }
.card-header p { color: var(--color-text-dim); margin: 0.5rem 0 0; }
.features { display: flex; justify-content: space-around; background-color: rgba(15, 23, 42, 0.5); padding: 1rem; border-radius: 16px; margin-bottom: 2rem; }
.feature-item { display: flex; flex-direction: column; align-items: center; gap: 0.5rem; color: var(--color-text-dim); transition: color 0.3s ease; }
.feature-item:hover { color: var(--color-text); }
.feature-item svg { width: 24px; height: 24px; }
.feature-item span { font-size: 0.875rem; font-weight: 500; }
.input-container textarea { width: 100%; background: rgba(15, 23, 42, 0.7); border: 1px solid var(--color-border); border-radius: 12px; padding: 1rem; color: var(--color-text); font-size: 1rem; line-height: 1.6; resize: none; transition: all 0.3s ease; }
.input-container textarea:focus { outline: none; border-color: var(--color-primary); box-shadow: 0 0 0 3px rgba(129, 140, 248, 0.3); }
.action-button { width: 100%; padding: 1rem; margin-top: 1.5rem; border: none; border-radius: 12px; background: linear-gradient(90deg, var(--color-primary), var(--color-aurora-2)); color: white; font-size: 1.125rem; font-weight: 600; cursor: pointer; transition: all 0.3s ease; box-shadow: 0 4px 15px rgba(129, 140, 248, 0.2); }
.action-button:hover:not(:disabled) { transform: translateY(-3px); box-shadow: 0 7px 20px rgba(129, 140, 248, 0.3); }
.action-button:disabled { background: #334155; cursor: not-allowed; box-shadow: none; }
.loading-state { display: flex; align-items: center; justify-content: center; gap: 0.75rem; }
.page-footer a { color: var(--color-primary); text-decoration: none; font-weight: 500; transition: color 0.3s ease; }
.page-footer a:hover { color: white; }
.spinner { animation: rotate 2s linear infinite; width: 20px; height: 20px; }
.path { stroke: white; stroke-linecap: round; animation: dash 1.5s ease-in-out infinite; }
@keyframes rotate { 100% { transform: rotate(360deg); } }
@keyframes dash { 0% { stroke-dasharray: 1, 150; stroke-dashoffset: 0; } 50% { stroke-dasharray: 90, 150; stroke-dashoffset: -35; } 100% { stroke-dasharray: 90, 150; stroke-dashoffset: -124; } }
@media (max-width: 768px) { .glass-card { padding: 1.5rem; } .card-header h1 { font-size: 1.75rem; } .features { flex-direction: column; gap: 1.5rem; } .result-card { width: calc(100% - 2rem); height: 85vh; border-radius: 16px; padding: 1.5rem;} }
</style>