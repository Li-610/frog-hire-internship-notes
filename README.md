# frog-hire-internship-notes

本仓库用于总结我在实习过程中对 **FrogHire.ai 浏览器扩展项目** 的学习和理解。  
目标是 **熟悉工具链、主要库、代码结构**，并在周会上汇报进展。

---

## 工具链概览

- **Node.js + npm/pnpm**  
  项目运行环境和包管理器，用于安装依赖和运行构建脚本。

- **WXT (Web eXtension Toolkit)**  
  浏览器扩展开发的核心框架。  
  - 自动生成 `manifest.json`  
  - 构建输出到 `.output/chrome-mv3/`（可在 Chrome 加载测试）  
  - 提供开发模式，支持热更新  
    [文档](https://wxt.dev)

- **Vite**  
  WXT 内部使用的快速打包工具，处理 TS/Vue/Tailwind。  
  [文档](https://vitejs.dev)

- **TypeScript**  
  为项目增加类型安全（例如 autofill 配置里的 `ApplicationConfig`）。  
  [文档](https://www.typescriptlang.org)

---

## 前端常用库

- **Vue 3** → 用于扩展的 UI（popup、options 页面）。  
- **TailwindCSS** → 原子化 CSS 框架，用 class 快速写样式。  
- **Nuxt UI Pro** → 基于 Tailwind 的 Vue 组件库。  

---

## 项目结构（关键部分）
