# frog-hire-internship-notes

本仓库用于总结我在实习过程中对 **FrogHire.ai 浏览器扩展项目** 的学习和理解。  

---

## 工具链概览

- **Node.js + npm/pnpm**  
  整个项目依赖 Node.js 运行，`pnpm` 用于安装和管理依赖。  
  项目脚本（`npm run dev` / `npm run build`）都依赖 Node.js 环境执行。  

- **WXT (Web eXtension Toolkit)**  
  核心开发框架，专门用于开发 Chrome/Firefox 扩展。  
  - 读取 `wxt.config.ts` 生成对应的 `manifest.json`  
  - 把代码构建成 `.output/chrome-mv3/`（可以在 Chrome 加载调试）  
  - 提供命令：`wxt`（开发模式）、`wxt build`（生产构建）、`wxt zip`（打包发布）  

- **Vite**  
  打包工具，WXT 内部就是基于 Vite。  
  - 处理 TypeScript 和 Vue 单文件组件（SFC）  
  - 和 Tailwind 一起运行，实现快速热更新  

- **TypeScript**  
  项目里大量用到类型定义，例如 `ApplicationConfig`。  
  - 确保 autofill 配置文件里的字段和规则合法  
  - 避免写错选择器或者 action 类型  

- **lefthook**  
  Git hook 工具，在提交代码时运行检查（例如 Biome 格式化）。  

---

## 前端框架与 UI 库

- **Vue 3**  
  用于实现扩展的 UI（popup、options 面板）。  
  - 使用组合式 API（`ref`、`reactive`、`computed`）管理状态  
  - 示例：选项页面可以用 Vue 渲染一个表单来管理 autofill 配置  

- **vue-router**  
  项目里如果扩展 UI 有多个页面（比如 popup 和设置页面），会用路由管理跳转。  

- **TailwindCSS**  
  工具类 CSS 框架，直接在 HTML/TSX 里写 `class="p-4 bg-gray-100"` 实现样式。  
  - 项目 popup 和 options 页面的布局和按钮都用 Tailwind 快速写出来  

- **Nuxt UI / Nuxt UI Pro**  
  Vue 组件库，基于 Tailwind 封装好的组件（比如按钮、卡片、输入框）。  
  - 可以直接用 `<UButton>`，不用自己写一堆样式  

- **@vueuse/core**  
  提供 Vue 常用的组合式函数，比如监听窗口大小、复制到剪贴板等。  
  - 在扩展 UI 里可以减少很多重复逻辑  

- **Iconify（lucide/simple-icons）**  
  图标库，用于在扩展 UI 里显示社交图标（GitHub/LinkedIn 等）。  

---

## 功能类库（和 autofill 直接相关）

- **jsonpath-plus**  
  用于从简历 JSON 数据里提取字段。  
  - 例如：`$.profile.basics.email` 就能取出邮箱  
  - 在 `lever.ts`、`workable.ts` 里 action 的 `value` 字段经常用到  

- **date-fns**  
  日期处理库，用来格式化和比较时间。  
  - 如果 autofill 表单里有“Available from date”，就会用这个库来生成日期字符串  

- **es-toolkit**  
  工具函数集合，类似 lodash。  
  - 例如数组去重、对象深拷贝，可以在 autofill 配置里写 transforms 用到  

- **@webext-core/messaging**  
  浏览器扩展里的消息通信模块。  
  - 背景页（background）和内容脚本（content script）之间通信时用到  

---

## 测试 & 质量保证

- **Vitest**  
  测试框架（类似 Jest，但基于 Vite，速度快）。  
  - 可以写单元测试来验证 autofill 规则是否能正确匹配 DOM  

- **Biome**  
  代码质量工具，集成了 Lint + 格式化。  
  - 代替了 ESLint + Prettier  
  - 在 `npm run check` 时运行，保证代码风格统一  

- **@types/node**  
  Node.js 类型定义，保证 TS 在写脚本时不会报错。  
