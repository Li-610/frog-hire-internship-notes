# frog-hire-internship-notes

本仓库用于总结我在实习过程中对 **FrogHire.ai 浏览器扩展项目** 的学习和理解。  

---

## 工具链

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

## 前端框架

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

## 库

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

## 测试

- **Vitest**  
  测试框架（类似 Jest，但基于 Vite，速度快）。  
  - 可以写单元测试来验证 autofill 规则是否能正确匹配 DOM  

- **Biome**  
  代码质量工具，集成了 Lint + 格式化。  
  - 代替了 ESLint + Prettier  
  - 在 `npm run check` 时运行，保证代码风格统一  

- **@types/node**  
  Node.js 类型定义，保证 TS 在写脚本时不会报错。  


---
title: "前端工程 Q&A 速查（单文件版）"
author: "Your Name"
date: "2025-10-02"
description: "pnpm / monorepo / Vite / TypeScript / Vue3 / Biome / Lefthook 常见问题与示例（一个文件包含全部内容，可直接复制为 README.md）"
---

# 前端工程 Q&A 速查（单文件版）

> 本文将以下问题集中在**一份** Markdown 文件中，便于复制使用：
>
> - **pnpm**：monorepo 与 workspace、共享模块
> - **Vite**：路径别名、环境配置、静态资源处理
> - **TypeScript**：基础类型、数组对象、联合交叉、工具类型、推断与类型保护、type vs interface
> - **Vue 3**：Composition API、生命周期、组件通信、模板语法、TS 集成、Composables 组织
> - **Biome**：用途与基本配置
> - **Lefthook**：基本配置与常用命令

---

## pnpm

### 什么是 monorepo
- **Monorepo**：使用**一个仓库**存放多个包/应用（packages/apps），便于**共享代码、统一规范与版本**、集中管理依赖与发布流程。
- 对比 **polyrepo**（多个仓库）：monorepo 更利于重用与协作，但需要工具（如 pnpm workspace、Nx、Turborepo）管理边界与性能。

### monorepo 项目里是怎么用 pnpm workspace 管理多个包依赖的
1. **根目录配置 `pnpm-workspace.yaml`**
   ~~~yaml
   packages:
     - "packages/*"
     - "apps/*"
   ~~~
2. **各包使用 `package.json` 声明名称与入口**
   ~~~json
   {
     "name": "@org/utils",
     "version": "0.1.0",
     "main": "dist/index.js"
   }
   ~~~
3. **内部依赖用 `workspace:` 协议**（自动联结本地包）
   ~~~json
   {
     "dependencies": {
       "@org/utils": "workspace:^"
     }
   }
   ~~~
4. **安装与构建**
   ~~~bash
   pnpm install
   pnpm -r build        # -r/--recursive：在所有工作区包里执行
   pnpm -r test
   ~~~
5. **过滤执行**
   ~~~bash
   pnpm -F @org/utils build       # 只在 @org/utils 包里执行
   pnpm -F "app-*" dev            # 在匹配到的多个包里执行
   ~~~
6. **去重与全局管理**
   - pnpm 使用**内容寻址**与**store**，在 monorepo 中对依赖去重显著。

### 在 monorepo 里，如何共享公共模块？
- 将公共代码放到 `packages/shared` 或 `packages/*`，以包形式暴露 API。
- 使用 `workspace:` 引用，保证本地开发联调。
- 若需跨包 TS 类型共享，可在公共包导出 d.ts，或使用 TS path 别名（见 Vite/TS 配置）。
- 若是样式/主题，可提供 `@org/theme` 包，导出 CSS/SCSS/设计 token。

---

## Vite

### Vite 项目里是怎么配置路径别名的？
1. **安装/使用 `path`（Node内置模块）并在 `vite.config.ts` 配置**：
   ~~~ts
   // vite.config.ts
   import { defineConfig } from "vite";
   import vue from "@vitejs/plugin-vue";
   import path from "node:path";

   export default defineConfig({
     plugins: [vue()],
     resolve: {
       alias: {
         "@": path.resolve(__dirname, "src"),
         "~utils": path.resolve(__dirname, "src/utils")
       }
     }
   });
   ~~~
2. **为 TypeScript 同步 alias（tsconfig.json / tsconfig.paths.json）**：
   ~~~json
   {
     "compilerOptions": {
       "baseUrl": ".",
       "paths": {
         "@/*": ["src/*"],
         "~utils/*": ["src/utils/*"]
       }
     }
   }
   ~~~
3. **使用示例**：
   ~~~ts
   import { foo } from "@/lib/foo";
   import { bar } from "~utils/bar";
   ~~~

### 怎么管理不同环境（开发/生产）的配置
- 使用 `.env` 系列文件：  
  - `.env`（所有环境）、`.env.development`、`.env.production`、`.env.test` …
  - 以 `VITE_` 前缀的变量会暴露到客户端代码中。
- 示例文件：
  ~~~bash
  # .env
  VITE_API_BASE=/api

  # .env.development
  VITE_API_BASE=http://localhost:3000

  # .env.production
  VITE_API_BASE=https://api.example.com
  ~~~
- 代码读取：
  ~~~ts
  const base = import.meta.env.VITE_API_BASE;
  ~~~
- 在 `vite.config.ts` 中可根据 `mode` 分支：
  ~~~ts
  export default defineConfig(({ mode }) => {
    const isProd = mode === "production";
    return {
      define: {
        __APP_VERSION__: JSON.stringify("1.0.0")
      },
      build: {
        sourcemap: !isProd
      }
    };
  });
  ~~~

### 静态资源（图片、字体等）在 Vite 项目里是怎么处理的
- `src` 内通过 `import` 引用，Vite 会进行打包处理（hash、优化）。
  ~~~ts
  import logoUrl from "@/assets/logo.svg";
  ~~~
- `public/` 目录下的文件会**原样复制**到输出根目录（通过绝对路径访问）：
  - `public/favicon.ico` → 打包后 `/favicon.ico`
  - 代码中直接使用：`<img src="/favicon.ico" />`
- CSS/字体等可通过 `url()`、`@font-face` 引用；小文件可能被内联为 base64，阈值可配置。

---

## TypeScript

> 文档：<https://www.learn-ts.org/>

### 基础类型有哪些
- `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`, `void`, `never`, `unknown`
- 字面量类型：`"GET" | "POST"`、 `1 | 2 | 3`

### 数组和对象类型：如何定义数组、对象、嵌套对象类型
- 数组：
  ~~~ts
  const arr1: number[] = [1, 2, 3];
  const arr2: Array<string> = ["a", "b"];
  ~~~
- 对象与嵌套：
  ~~~ts
  type Address = { city: string; zip?: string };
  interface User {
    id: number;
    name: string;
    address: Address;        // 嵌套对象
    tags?: string[];         // 可选属性
  }
  ~~~

### 联合类型 & 交叉类型：常见场景
- **联合**（一个值可为多种类型）：接口返回字段可能是字符串或数字
  ~~~ts
  type ID = string | number;

  function getUser(id: ID) { /* ... */ }
  ~~~
- **交叉**（组合多个对象类型）：合并配置
  ~~~ts
  type Base = { id: string };
  type WithTime = { createdAt: Date };
  type Entity = Base & WithTime;     // 同时具有两者属性
  ~~~

### 工具类型：Partial、Pick、Omit、Record 等在项目中如何用
- `Partial<T>`：将 T 的所有属性设为可选（用于**更新**场景）
  ~~~ts
  type User = { id: number; name: string; email: string };
  type UserPatch = Partial<User>;
  ~~~
- `Pick<T, K>`：从 T 中**挑选**部分属性
  ~~~ts
  type UserView = Pick<User, "id" | "name">;
  ~~~
- `Omit<T, K>`：从 T 中**排除**某些属性
  ~~~ts
  type UserWithoutEmail = Omit<User, "email">;
  ~~~
- `Record<K, T>`：构造键值映射
  ~~~ts
  type LocaleMap = Record<"en" | "zh" | "jp", string>;
  ~~~

### 类型推断与类型保护
- 推断：TS 会根据初始值推断类型，尽量**少用 `any`**。
- 类型保护（Type Guards）：
  ~~~ts
  function isString(x: unknown): x is string {
    return typeof x === "string";
  }

  function print(x: unknown) {
    if (isString(x)) {
      console.log(x.toUpperCase()); // 经过保护后，x 被视为 string
    }
  }
  ~~~

### type 和 interface 的区别，如何选择
- 共性：都能描述对象、函数、类结构；都支持扩展与合并（`interface` 原生合并声明更灵活）。
- 差异：
  - `interface` 可**重复声明自动合并**；`type` 不行。
  - `type` 可用于**联合/交叉**、条件类型等更复杂的类型表达。
- 选择建议：
  - **对象/类的结构**优先用 `interface`（可拓展、合并友好）。
  - **联合类型/工具类型复合**优先用 `type`。

---

## Vue 3

### Composition API 有哪些，区别，使用场景
- **核心**：`ref`, `reactive`, `computed`, `watch`, `watchEffect`
- **组件相关**：`defineProps`, `defineEmits`, `defineExpose`, `defineSlots`
- **生命周期**：`onMounted`, `onUpdated`, `onUnmounted`, `onBeforeMount`, `onBeforeUpdate`, `onBeforeUnmount`, `onActivated`, `onDeactivated`, `onErrorCaptured`
- **场景**：
  - `ref`：基本类型响应式或单值容器
  - `reactive`：对象/数组深层响应式
  - `computed`：基于依赖自动缓存的派生值
  - `watch`：对特定源变化执行副作用（需要精确控制时用）
  - `watchEffect`：自动收集依赖的副作用（快速/粗粒度）

### 生命周期（常用）
- `onMounted`：DOM 已挂载，适合发起请求、操作 DOM
- `onUpdated`：响应式更新导致 DOM 变更后
- `onUnmounted`：组件卸载时清理资源
- 还有 `onBefore*`、`onErrorCaptured`、`onActivated/Deactivated`（KeepAlive）

### 组件通信
- 父 → 子：**props**
- 子 → 父：**emits**（`defineEmits`/`$emit`）
- 祖先 ↔ 后代：**provide/inject**
- 双向绑定：**`v-model`**（可自定义 prop 与事件）
- 跨组件共享：**全局状态（Pinia）** 或事件总线（不推荐大型项目）

### 模板语法（常见）
- 插值：`{{ value }}`
- 属性绑定：`:src="imgUrl"` / `v-bind:src="..."`
- 事件：`@click="handleClick"` / `v-on:click="..."`
- 条件与列表：`v-if / v-else-if / v-else`，`v-for="(item, i) in list" :key="i"`
- 类与样式：`:class="{ active: isActive }"`，`:style="{ color: themeColor }"`

### TypeScript 集成
- 在 `script setup` 中直接打类型：
  ~~~vue
  <script setup lang="ts">
  const props = defineProps<{
    id: string
    count?: number
  }>();

  const emit = defineEmits<{
    (e: "update:count", value: number): void
  }>();
  </script>
  ~~~
- 启用 `vue-tsc` 校验，配合 Volar 插件提升类型体验。

### 项目里如何组织和复用组合式函数（Composables）
- 约定放在 `src/composables/`，命名 `useXxx.ts`
- 保持**单一职责**，输入/返回类型明确，尽量无副作用（或可控）。
  ~~~ts
  // src/composables/useFetch.ts
  import { ref } from "vue";

  export function useFetch<T>(url: string) {
    const data = ref<T | null>(null);
    const loading = ref(false);
    const error = ref<Error | null>(null);

    const run = async () => {
      loading.value = true;
      try {
        const res = await fetch(url);
        data.value = await res.json();
      } catch (e) {
        error.value = e as Error;
      } finally {
        loading.value = false;
      }
    };

    return { data, loading, error, run };
  }
  ~~~

---

## Biome

### 作用是什么
- **Biome** 是一个**现代化、极速**的前端工具链，集 **Formatter（格式化）** + **Linter（代码质量）** + **Import 排序** 等于一体（目标是替代 Prettier + ESLint 的常见组合），内置性能优化（Rust 实现），上手成本低。

### 了解基本配置规则
1. 安装（示例）
   ~~~bash
   pnpm add -D @biomejs/biome
   ~~~
2. 配置文件 `biome.json`（或 `biome.jsonc`）
   ~~~json
   {
     "$schema": "https://biomejs.dev/schemas/1.8.0/schema.json",
     "formatter": {
       "enabled": true,
       "lineWidth": 100
     },
     "linter": {
       "enabled": true,
       "rules": {
         "style": {
           "useConst": "error"
         },
         "correctness": {
           "noUnusedVariables": "warn"
         }
       }
     },
     "organizeImports": {
       "enabled": true
     }
   }
   ~~~
3. 常用命令
   ~~~bash
   npx biome check .           # 代码质量/格式检查
   npx biome format . --write  # 自动格式化
   npx biome ci .              # CI 模式
   ~~~

---

## Lefthook

### 基本配置和命令
- **Lefthook** 是一个**快速、并行**、跨平台的 Git hooks 管理工具，替代 Husky 等。
1. 安装
   ~~~bash
   pnpm add -D lefthook
   npx lefthook install        # 安装 git hooks
   ~~~
2. 配置 `lefthook.yml`
   ~~~yaml
   pre-commit:
     parallel: true
     commands:
       biome:
         run: npx biome check .
       typecheck:
         run: pnpm -w typecheck
       tests:
         run: pnpm -w test -i
   pre-push:
     commands:
       build:
         run: pnpm -w -r build
   ~~~
   - `-w`：在 workspace 根执行
   - `-r`：递归执行所有包
3. 常用命令
   ~~~bash
   npx lefthook run pre-commit
   npx lefthook run pre-push
   ~~~
4. 与 monorepo 配合
   - 在根配置一次，利用 `-w` 和 `-r` 覆盖全项目。
   - 可结合 `pnpm -F` 做**按包过滤**，如只检查变更的包（结合变更检测脚本）。

---

## 附：综合小示例（monorepo + Vite + Vue3 + TS）

**目录结构**
~~~text
.
├─ apps/
│  └─ web/                    # Vite + Vue3 应用
├─ packages/
│  ├─ utils/                  # TS 工具库
│  └─ theme/                  # 共享样式/设计 token
├─ pnpm-workspace.yaml
└─ lefthook.yml
~~~

**`pnpm-workspace.yaml`**
~~~yaml
packages:
  - "apps/*"
  - "packages/*"
~~~

**`apps/web/vite.config.ts`**
~~~ts
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import path from "node:path";

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "src"),
      "@utils": path.resolve(__dirname, "../../packages/utils/src")
    }
  }
});
~~~

**`apps/web/.env.development`**
~~~bash
VITE_API_BASE=http://localhost:3000
~~~

**`packages/utils/src/index.ts`**
~~~ts
export const sum = (a: number, b: number) => a + b;
~~~

**`apps/web/src/App.vue`**
~~~vue
<script setup lang="ts">
import { ref, onMounted } from "vue";
import { sum } from "@utils";

const count = ref(0);
onMounted(() => {
  count.value = sum(1, 2);
});
</script>

<template>
  <h1>Count: {{ count }}</h1>
</template>
~~~

---

