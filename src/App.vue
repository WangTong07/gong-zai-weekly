<script setup>
import { ref } from 'vue';

// 定义响应式变量来存储用户输入、加载状态和结果
const userInput = ref('');
const loading = ref(false);
const reportResult = ref('');

// 【已修改】直接从Vercel的环境变量中读取API Key
// 注意：这个变量名 VITE_ZHIPU_API_KEY 必须和你在Vercel上配置的完全一致
const apiKey = import.meta.env.VITE_ZHIPU_API_KEY;

// 定义生成周报的异步函数
async function generateReport() {
  if (!userInput.value.trim()) {
    alert('请输入你的工作内容！');
    return;
  }
  // 【已移除】不再需要检查用户是否输入API Key

  loading.value = true;
  reportResult.value = '';

  const prompt = `你是一名资深项目经理，请将以下我的本周工作记录，整理成一份专业、正式、向上汇报的周报。请分点阐述，并适当润色，突出工作亮点和价值。周报需要包含以下几个部分：1. 本周重点工作总结 2. 主要成果与数据 3. 下周工作计划。我的工作记录是：『${userInput.value}』`;

  try {
    const response = await fetch("https://open.bigmodel.cn/api/paas/v4/chat/completions", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        // 【已修改】直接使用apiKey变量，不再需要.value
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
        // 显示更友好的错误提示
        alert(`请求失败: ${errorBody.error?.message || response.statusText}`);
        throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    reportResult.value = data.choices[0].message.content;

  } catch (error) {
    console.error("请求AI API失败:", error);
    if (!alert.toString().includes('请求失败')) { // 避免重复弹窗
        alert('生成报告时遇到问题，请稍后再试或查看控制台获取详情。');
    }
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <main>
    <div class="container">
      <h1>我是一个“打工仔”周报生成器 🤖</h1>
      <p class="description">
        把本周的琐事流水账丢进来，AI帮你生成一份体面的周报。
      </p>

      <!-- 【已移除】API Key的输入框整个部分都删掉了 -->

      <textarea
        v-model="userInput"
        rows="10"
        placeholder="例如：
 - 周一和产品经理开了个需求会对了下细节
 - 周二修复了3个线上bug
 - 写了XX新功能的开发文档
 - 帮同事小王解决了一个技术难题"
      ></textarea>

      <button @click="generateReport" :disabled="loading">
        {{ loading ? '正在拼命生成中...' : '一键生成周报' }}
      </button>

      <div v-if="reportResult" class="result-container">
        <h2>✨ AI生成结果 ✨</h2>
        <pre class="result-text">{{ reportResult }}</pre>
      </div>
    </div>
  </main>
</template>

<style scoped>
main {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: flex;
  justify-content: center;
  padding: 2rem;
  background-color: #f7fafc;
  min-height: 100vh;
}
.container {
  width: 100%;
  max-width: 768px;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}
h1 {
  font-size: 2.25rem;
  font-weight: bold;
  text-align: center;
  color: #2d3748;
}
.description {
  text-align: center;
  color: #718096;
  font-size: 1.1rem;
}
textarea {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid #e2e8f0;
  border-radius: 0.375rem;
  font-size: 1rem;
  box-sizing: border-box;
  resize: vertical;
}
button {
  padding: 0.75rem 1.5rem;
  background-color: #4299e1;
  color: white;
  font-weight: bold;
  border: none;
  border-radius: 0.375rem;
  cursor: pointer;
  transition: background-color 0.2s;
  font-size: 1rem;
}
button:hover:not(:disabled) {
  background-color: #3182ce;
}
button:disabled {
  background-color: #a0aec0;
  cursor: not-allowed;
}
.result-container {
  margin-top: 1rem;
  padding: 1.5rem;
  background-color: white;
  border: 1px solid #e2e8f0;
  border-radius: 0.5rem;
  box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px 0 rgba(0, 0, 0, 0.06);
}
.result-text {
  white-space: pre-wrap; /* 自动换行 */
  word-wrap: break-word;
  color: #4a5568;
  font-size: 1rem;
  line-height: 1.6;
}
</style>