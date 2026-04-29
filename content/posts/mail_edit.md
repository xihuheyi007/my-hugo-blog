---
date: '2026-04-30T01:41:24+08:00'
draft: false
title: 'Cloud-Mail UI 优化实践'
tags: ["技术"]
categories: ["兴趣使然"]
---
## 前言

Cloud Mail 是一个基于 Cloudflare Workers 的邮箱服务项目，本文记录了对该项目的 UI 优化实践，包括主题系统升级、侧边栏改造和深色模式适配。

>本文使用Minimax-2.7模型生成
---

## 一、主题系统升级

### 1.1 问题

原项目仅支持浅色/深色手动切换，无法跟随系统偏好，且状态管理混乱。

### 1.2 解决方案

**新增 `useTheme` Composable**

```javascript
// src/composables/useTheme.js
export function useTheme() {
  const uiStore = useUiStore()
  let mediaQuery = null

  const applyTheme = () => {
    const isDark = uiStore.isDark
    document.documentElement.classList.toggle('dark', isDark)
  }

  onMounted(() => {
    mediaQuery = window.matchMedia('(prefers-color-scheme: dark)')
    uiStore.updateSystemTheme(mediaQuery.matches)
    mediaQuery.addEventListener('change', (e) => {
      uiStore.updateSystemTheme(e.matches)
    })
    applyTheme()
  })

  watch(() => uiStore.isDark, applyTheme)
}
```

**重构状态管理**

```javascript
// src/store/ui.js
state: () => ({
  themeMode: 'system',  // 'light' | 'dark' | 'system'
  systemPrefersDark: window.matchMedia('(prefers-color-scheme: dark)').matches
}),
getters: {
  isDark: (state) => {
    if (state.themeMode === 'system') return state.systemPrefersDark
    return state.themeMode === 'dark'
  }
}
```

### 1.3 效果

- 支持浅色、深色、跟随系统三种模式
- 系统主题变化时实时响应
- 用户偏好自动持久化

---

## 二、侧边栏优化

### 2.1 问题

原设计折叠后仅缩小为 60px 图标模式，仍占用空间影响布局。

### 2.2 解决方案

**分离桌面端与移动端逻辑**

```javascript
// src/layout/index.vue
const asideClass = computed(() => {
  if (isMobile.value) {
    return uiStore.asideShow ? 'aside-drawer-open' : 'aside-drawer-hide'
  }
  return uiStore.asideCollapsed ? 'aside-collapsed' : 'aside-expanded'
})
```

**使用 Transform 动画替代 Width 动画**

```scss
.aside-collapsed {
  transform: translateX(-100%);
  width: 0 !important;
}

.aside-expanded {
  transform: translateX(0);
  width: 260px;
  transition: transform 150ms ease;
}
```

### 2.3 效果

| 对比项 | 优化前 | 优化后 |
|--------|--------|--------|
| 折叠状态 | 60px 图标残留 | 完全隐藏 |
| 动画方式 | width 变化 | transform GPU 加速 |
| 动画时长 | 200ms | 150ms |

---

## 三、CSS 变量化改造

### 3.1 问题

硬编码颜色散落在各处，主题切换时无法统一响应。

### 3.2 解决方案

**定义完整的主题变量系统**

```scss
:root {
  --aside-background: #ffffff;
  --aside-menu-text: #1f1f1f;
  --aside-hover-bg: #f0f2f5;
  --aside-active-bg: rgba(24, 144, 255, 0.1);
}

.dark {
  --aside-background: #141414;
  --aside-menu-text: #ffffff;
  --aside-hover-bg: #2a2a2a;
  --aside-active-bg: rgba(255, 255, 255, 0.08);
}
```

**组件中使用变量**

```vue
<el-menu :background-color="menuBgColor">
<!--
  menuBgColor = computed(() => 'var(--aside-background)')
-->
```

### 3.3 效果

- 全部颜色值通过变量统一管理
- 主题切换时自动响应
- 代码可维护性大幅提升

---

## 四、深色模式邮件内容适配

### 4.1 问题

邮件正文使用 Shadow DOM 渲染，原有样式硬编码白色背景，无法适配深色模式。

### 4.2 解决方案

**传递 `dark` prop 动态设置样式**

```javascript
// src/components/shadow-html/index.vue
const props = defineProps({
  html: { type: String, required: true },
  dark: { type: Boolean, default: false }
})

function updateContent() {
  const bgColor = props.dark ? '#1a1a1a' : '#FFFFFF'
  const textColor = props.dark ? '#e0e0e0' : '#13181D'
  const linkColor = props.dark ? '#6db3f2' : '#0E70DF'
  // ... 动态生成样式
}
```

**父组件传递状态**

```vue
<ShadowHtml :html="content" :dark="uiStore.isDark" />
```

### 4.3 深色配色方案

| 元素 | 浅色 | 深色 |
|------|------|------|
| 背景 | #FFFFFF | #1a1a1a |
| 文字 | #13181D | #e0e0e0 |
| 链接 | #0E70DF | #6db3f2 |

---

## 五、修改文件清单

| 文件 | 修改类型 | 说明 |
|------|----------|------|
| src/store/ui.js | 重构 | 新增 themeMode 状态和 isDark getter |
| src/composables/useTheme.js | 新增 | 统一管理主题逻辑 |
| src/App.vue | 修改 | 引入 useTheme |
| src/layout/index.vue | 修改 | 侧边栏折叠逻辑优化 |
| src/layout/aside/index.vue | 修改 | CSS 变量化 |
| src/layout/header/index.vue | 修改 | 主题选择下拉菜单 |
| src/style.css | 修改 | 主题变量定义、卡片样式 |
| src/components/shadow-html/index.vue | 修改 | 深色模式支持 |
| src/views/content/index.vue | 修改 | 传递 dark prop |
| src/i18n/zh.js | 修改 | 新增主题相关翻译 |
| src/i18n/en.js | 修改 | 新增主题相关翻译 |

---

## 六、总结

本次优化实现了：

1. **主题系统升级** - 三种模式 + 系统跟随 + 持久化
2. **侧边栏改造** - 完全隐藏 + GPU 加速动画
3. **CSS 架构优化** - 变量化 + 统一管理
4. **深色模式完善** - Shadow DOM 内容适配

所有修改均保持了原有功能不变，仅涉及 UI 层。
