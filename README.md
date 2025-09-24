# frog-hire-internship-notes

本仓库用于总结我在实习过程中对 **FrogHire.ai 浏览器扩展项目** 的学习和理解。  

---

## 工具链概览

- **Node.js + npm/pnpm**  
  项目运行环境和包管理器，用于安装依赖和运行构建脚本。

- **WXT (Web eXtension Toolkit)**  
  浏览器扩展开发的核心框架。  
  - 自动生成 `manifest.json`  
  - 构建输出到 `.output/chrome-mv3/`（可在 Chrome 加载测试）  
  - 提供开发模式，支持热更新  

- **Vite**  
  WXT 内部使用的快速打包工具，处理 TS/Vue/Tailwind。  

- **TypeScript**  
  为项目增加类型安全（例如 autofill 配置里的 `ApplicationConfig`）。  
  - 学习了 `type` 和 `interface` 的区别  
  - 熟悉了 import/export 的用法  
  - 理解了如何用类型来约束 autofill 配置结构  

---

## 前端常用库

- **Vue 3**  
  - 使用组合式 API（`ref`、`reactive`、`computed`）管理状态  
  - 在扩展 UI（popup、options）中使用  
- **TailwindCSS**  
  - 原子化 CSS 框架  
  - 用 class 快速实现布局和样式  
- **Nuxt UI Pro**  
  - 基于 Tailwind 的 Vue 组件库  
  - 提供现成的 UI 组件提升开发效率  


---

- **工具链 / 构建相关**  
  `wxt`, `typescript`, `tailwindcss`, `@wxt-dev/module-vue`, `vue-tsc`, `lefthook`  

- **前端框架 / UI**  
  `vue`, `vue-router`, `@nuxt/ui`, `@nuxt/ui-pro`, `@vueuse/core`,  
  `@iconify-json/lucide`, `@iconify-json/simple-icons`  

- **功能类库**  
  `jsonpath-plus`, `date-fns`, `es-toolkit`, `@webext-core/messaging`  

- **测试 & 质量**  
  `vitest`, `@biomejs/biome`, `@types/node`  
