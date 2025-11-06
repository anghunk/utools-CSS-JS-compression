<script setup>
import { ref, computed } from "vue";
const activeTab = ref("css");
const inputCode = ref("");
const outputCode = ref("");
const toastVisible = ref(false);
const toastMessage = ref("");
const toastType = ref("success");
let toastTimer = null;
const showToast = (message, type = "success", duration = 2000) => {
  toastMessage.value = message;
  toastType.value = type;
  toastVisible.value = true;
  if (toastTimer) {
    clearTimeout(toastTimer);
  }
  toastTimer = setTimeout(() => {
    toastVisible.value = false;
  }, duration);
};

// 切换tab
const switchTab = (tab) => {
  activeTab.value = tab;
  inputCode.value = "";
  outputCode.value = "";
};

// CSS压缩逻辑
const compressCSS = (code) => {
  return (
    code
      // 移除注释
      .replace(/\/\*[\s\S]*?\*\//g, "")
      // 移除多余空格
      .replace(/\s+/g, " ")
      // 移除冒号后的空格
      .replace(/\s*:\s*/g, ":")
      // 移除分号前的空格
      .replace(/\s*;\s*/g, ";")
      // 移除大括号周围的空格
      .replace(/\s*\{\s*/g, "{")
      .replace(/\s*\}\s*/g, "}")
      // 移除逗号后的空格
      .replace(/\s*,\s*/g, ",")
      // 移除大于号周围的空格
      .replace(/\s*>\s*/g, ">")
      // 移除加号周围的空格（CSS选择器）
      .replace(/\s*\+\s*/g, "+")
      // 移除波浪号周围的空格
      .replace(/\s*~\s*/g, "~")
      // 移除开头和结尾的空格
      .trim()
  );
};

// JS压缩逻辑
const compressJS = (code) => {
  return (
    code
      // 移除单行注释
      .replace(/\/\/.*$/gm, "")
      // 移除多行注释
      .replace(/\/\*[\s\S]*?\*\//g, "")
      // 处理每一行
      .split("\n")
      .map((line) => line.trim())
      .filter((line) => line.length > 0)
      .map((line) => {
        // 如果行末没有分号、花括号、冒号，且不是控制语句，则添加分号
        const lastChar = line.charAt(line.length - 1);
        const needsSemicolon =
          ![";", "{", "}", ":", ","].includes(lastChar) &&
          !line.match(
            /^\s*(if|else|for|while|do|switch|case|default|function|class)\s*\(/
          ) &&
          !line.match(/^\s*(else|try|catch|finally|do)\s*$/);

        return needsSemicolon ? line + ";" : line;
      })
      .join(" ")
      // 移除运算符周围多余的空格
      .replace(/\s*([+\-*/%=<>!&|?:,;{}()\[\]])\s*/g, "$1")
      // 在某些运算符后添加必要的空格（避免语法错误）
      .replace(
        /(return|var|let|const|function|if|else|for|while|do|switch|case|break|continue|throw|try|catch|finally|new|delete|typeof|void|in|instanceof)\(/g,
        "$1 ("
      )
      .replace(/\)if\(/g, ") if(")
      .replace(/\)else/g, ") else")
      .replace(/\)for\(/g, ") for(")
      .replace(/\)while\(/g, ") while(")
      .replace(/;if\(/g, "; if(")
      .replace(/;for\(/g, "; for(")
      .replace(/;while\(/g, "; while(")
      .replace(/}else/g, "} else")
      .replace(/}catch/g, "} catch")
      .replace(/}finally/g, "} finally")
      // 清理多余的分号
      .replace(/;+/g, ";")
      .replace(/;}/g, "}")
      .replace(/;\)/g, ")")
      // 移除开头和结尾的空格
      .trim()
  );
};

// HTML压缩逻辑
const compressHTML = (code) => {
  return (
    code
      // 移除HTML注释
      .replace(/<!--[\s\S]*?-->/g, "")
      // 移除标签间的空白
      .replace(/>\s+</g, "><")
      // 移除多余的空格
      .replace(/\s+/g, " ")
      // 移除标签内属性周围多余的空格
      .replace(/\s*=\s*/g, "=")
      // 移除开头和结尾的空格
      .trim()
  );
};

// 压缩代码
const compress = () => {
  if (!inputCode.value.trim()) {
    outputCode.value = "";
    return;
  }

  switch (activeTab.value) {
    case "css":
      outputCode.value = compressCSS(inputCode.value);
      break;
    case "js":
      outputCode.value = compressJS(inputCode.value);
      break;
    case "html":
      outputCode.value = compressHTML(inputCode.value);
      break;
  }
};

// 复制到剪贴板
const copyToClipboard = () => {
  if (outputCode.value) {
    navigator.clipboard
      .writeText(outputCode.value)
      .then(() => {
        showToast("复制成功！", "success");
      })
      .catch(() => {
        showToast("复制失败，请手动复制", "error");
      });
  }
};
</script>

<template>
  <div class="compress-container">
    <!-- Toast提示 -->
    <transition name="toast-fade">
      <div v-if="toastVisible" :class="['toast', `toast-${toastType}`]">
        <span class="toast-icon">
          <span v-if="toastType === 'success'">✓</span>
          <span v-else-if="toastType === 'error'">✕</span>
          <span v-else-if="toastType === 'warning'">!</span>
          <span v-else>ℹ</span>
        </span>
        <span class="toast-message">{{ toastMessage }}</span>
      </div>
    </transition>

    <!-- Tab切换 -->
    <div class="tabs">
      <button :class="['tab-button', { active: activeTab === 'css' }]" @click="switchTab('css')">
        CSS
      </button>
      <button :class="['tab-button', { active: activeTab === 'js' }]" @click="switchTab('js')">
        JavaScript
      </button>
      <button :class="['tab-button', { active: activeTab === 'html' }]" @click="switchTab('html')">
        HTML
      </button>
    </div>

    <!-- 输入输出区域 -->
    <div class="content">
      <div class="input-section">
        <textarea v-model="inputCode" @input="compress" :placeholder="`请输入${activeTab.toUpperCase()}代码...`"
          class="code-textarea"></textarea>
      </div>

      <div class="input-section">
        <div class="section-header">
          <div class="actions">
            <button @click="copyToClipboard" class="action-btn" :disabled="!outputCode">
              📋 复制
            </button>
          </div>
        </div>
        <textarea v-model="outputCode" readonly placeholder="压缩后的代码将显示在这里..." class="code-textarea output"></textarea>
      </div>
    </div>

    <!-- 统计信息 -->
    <div class="stats" v-if="outputCode">
      <span>原始大小: {{ inputCode.length }} 字符</span>
      <span>压缩后: {{ outputCode.length }} 字符</span>
      <span>压缩比例:
        {{
          ((1 - outputCode.length / inputCode.length) * 100).toFixed(2)
        }}%</span>
    </div>
  </div>
</template>
