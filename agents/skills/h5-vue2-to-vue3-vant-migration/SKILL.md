---
name: h5-vue2-to-vue3-vant-migration
description: >-
  Vue2 H5→Vue3+Vant+Vite。迁入本仓库：views、router 分模块、拆文件 SFC、service DEV Mock、组件复用/迁入 src/components、禁默换 van-*；须 Todo 列出本仓已有组件核对、待迁入组件、每接口 Mock（见 SKILL「迁入本仓库：强制项」）。
---

# H5 页面迁移：Vue 2 → Vue 3 + Vant 升级

**依赖版本**：迁入本 Skill **所在仓库**以根目录 `package.json` 为准；全仓升级且不对齐本仓时，再对照下方「与 sx-integration-h5-grid 对齐」清单。

## 使用方式

阶段顺序：**Mock（含接口 DEV Mock）→ 组件/依赖收敛 → Vue3/Vant 改造**。未定义数据契约与任务列表前，不得大面积删旧请求。速查：[references/vue-vant-cheatsheet.md](references/vue-vant-cheatsheet.md)。

## 迁入本仓库：强制项（须写入任务列表）

Agent / Builder **必须**用可勾选任务（如 Cursor Todo）跟踪下列项，**禁止只在脑中清点**：

1. **组件（含本仓已有）**
   - 解析旧页模板、`index.json` 的 `usingComponents`、脚本 `import`，列出**每一个**用到的自定义/业务组件。
   - 在 `src/components`（及 `@/components`）检索**已有**实现：每命中一个，任务列表**单列一条**（例：「核对并接入 `ScrollList.vue`，对齐 props/插槽/事件」），完成前不得把该页改成 Vant 原子组件替代。
   - 本仓无、旧工程有：单列「迁入 `components/<name>/` 三合一 → `*.vue`」。
   - **防重复迁入**：迁**下一页**前，对清单中每个业务组件在 `src/components` **全路径/关键词检索**（含旧工程命名如 `scrollList`、`search`）。本仓**已有同职责实现则禁止再拷一份 SFC**；优先扩展现有组件（可选 props / 插槽 / emit），合并后删除冗余副本。确不可共用而必须第二份时，任务里须写**理由**，且文件名/路径须体现差异（避免后人再迁第三份）。
2. **接口**
   - 本迁移范围内**每新增或每改动**的 `src/service/*.ts` 导出函数：**严格照抄** [references/vue-vant-cheatsheet.md](references/vue-vant-cheatsheet.md)「**Service 层 Mock 模板（唯一规范）**」代码块结构（`useAxiosStore` + DEV `Promise.resolve({ code, msg, data })` + `axios.post`），仅替换函数名与 URL/`data`；**无 Mock 不得勾「接口完成」**。
3. **拆文件与样式**
   - 旧壳页面/组件：`index.less` 的 `@import` 链单列检查项，防漏样式。

上述与下文「本仓库：老页面迁入」细则一致；冲突时以**任务列表可勾选**为准执行。

## 本仓库：老页面迁入已有 Vue3 工程（固定规范）

将旧 H5 / 多页业务**挪入**当前 Vite + Vue 3 仓库时（`src/views`、`src/router`、`src/service`）。

1. **拆文件 → SFC**
   - 页面：`index.html` + `Page` 的 `index.js` + `index.less` + `index.json` → 一个 `index.vue`；导航与 `usingComponents` → 路由 `meta` + Vant 按需/`unplugin-vue-components`。
   - 组件：`index.html` + `Component` 的 `index.js` + `index.less` → 一个 `*.vue`（大 less 可旁挂再 `@import`，交付不保留旧三件套）。
   - `index.less` 的 `@import` 链须整体迁入或抽公共样式，禁止只抄 `@import` 以下片段。
   - `vue-infinite-scroll` 等旧指令按目标栈用组合式 / `IntersectionObserver` 等重写。

2. **路由**：`src/router/gridman.ts` 等子模块增加 `RouteRecordRaw` + 懒加载；`index.ts` 已合并子路由则通常只改子文件。

3. **依赖与组件策略**
   - assets、stores、utils、service 按需迁入，**优先复用**本仓；去掉 `$set` 等 Vue2 API，Toast/Router 对齐本仓。
   - **禁止**仅因装了 Vant 就用 `van-search` / `van-list` / `van-tabs` 等默换 `search`、`scrollList`、`tabsList` 等业务封装。顺序：**(1)** 本仓 `src/components` 已有 → 复用（**须进任务列表**）；**(2)** 旧工程迁入三合一；**(3)** 确为薄 Vant 封装或产品要求改版 → 可换 Vant，**须写理由**。Skill §4 管 **Vant 2→4 API**，不管「用 Vant 取代业务组件」。
   - **多页迁移时**：同一业务组件在仓库内只保留**一份真源**；后续页面只改引用路径，不重复迁入；若已出现双份，应合并为一并统一 import（见上「防重复迁入」）。

4. **Mock（service 内）**
   - 每迁接口：**仅**允许 cheatsheet「**Service 层 Mock 模板（唯一规范）**」中的写法；`data` 与旧页 `res.data` 同形；注释 `// 开发环境使用模拟数据` 或 `// mock: ...`。

5. **上下游路由**：跳转链上页面须同时有路由或显式改指向，避免断链。

## 与 sx-integration-h5-grid 对齐的目标版本（跨仓参考）

**本仓以 `package.json` 为准**；下列供**无本仓文件时**对照 grid：

**dependencies**：`vue` ^3.5.24、`vue-router` ^4.6.4、`vant` ^4.9.22、`pinia` ^3.0.4、`pinia-plugin-persistedstate` ^4.7.1、`axios` ^1.8.3、`less` ^4.5.1、`js-audio-recorder` ^1.0.7（按需）

**devDependencies**：`vite` ^7.2.4、`@vitejs/plugin-vue` ^6.0.1、`typescript` ~5.9.3、`vue-tsc` ^3.1.4、`@vue/tsconfig` ^0.8.1、`@types/node` ^24.10.1、`@types/terser` ^3.12.0、`terser` ^5.46.0、`autoprefixer` ^10.4.27、`postcss-px-to-viewport-8-plugin` ^1.2.5、`husky` ^9.1.7

**工程元数据**：`"type": "module"`、`"private": true`；脚本 `dev`/`build`/`preview`/`prepare` 见 grid。**Node**：满足 Vite 7 的 `engines`（常见 20.19+ / 22.12+）。

## 0. 迁移前盘点（必做）

- [ ] 版本与入口：`vue`/`router`/`pinia`/`vant`/构建链；`main`、全局插件与 `globalProperties`。
- [ ] 高风险：`filter`、`$set`、`$on`/`$off`、`slot-scope`、`beforeDestroy` 等（见 cheatsheet）。
- [ ] H5：`viewport`、`vw`/`rem`、微信 WebView、废弃 polyfill。
- [ ] 拆文件壳：`index.html`/`js`/`less`（及 `index.json`）；`index.less` 全部 `@import`。
- [ ] **任务列表草稿**：本页组件清单 + 本仓 `src/components` 命中项 + 待迁接口列表（每项对应 Mock）。
- [ ] **防重复**：对上述组件清单逐项在 `src/components` 内检索，确认无「同职责双份 SFC」后再迁入或新建。

## 1. Mock 与数据契约（与强制项一致）

1. 按页面列契约：列表/详情/分页/枚举/空错态；对照旧请求与模板字段。
2. **迁入本仓**：接口 Mock **仅**允许 cheatsheet「Service 层 Mock 模板（唯一规范）」代码块形态；列表 Mock ≥3–5 条含边界。
3. 静态 `src/mocks/` 或 MSW 为可选补充，不替代「每迁接口必有 DEV Mock」。
4. 自检：断网可验主分支；Mock 字段名与旧 `data`/模板一致。

## 2. 依赖与构建

Node 与包：见上「跨仓参考」；**视口 PostCSS 包名以本仓 `package.json` 为准**（可与 grid 的 `8-plugin` 不同）。入口：`createApp` + `use(router)` + `use(pinia)`；全局能力用 `globalProperties` / `provide`。

## 3. Vue 3 代码层要点

过滤器删除；`$set` 删除；`beforeDestroy`→`beforeUnmount`；`$on`→mitt/Pinia；`v-model`/插槽按官方迁移核对。详表见 cheatsheet。

## 4. Vant 2 → 4（^4.9.22）

安装与按需（`unplugin-vue-components` + `VantResolver`）；主题 CSS 变量；`Toast`/`Dialog` 函数式路径与真机点验。破坏性改名见 [Vant 升级指南](https://vant-ui.github.io/vant/#/zh-CN/migrate-from-v2)。

## 5. 路由

本仓：`src/router/gridman.ts` 等。通用：`createRouter` + `createWebHashHistory`/`createWebHistory`；`router.isReady()`；懒加载 `import()`。

## 6. 样式

`::v-deep`→`:deep()`；视口方案随本仓配置；安全区与 `NavBar`/`Tabbar`/`z-index` 手测。

## 7. 验证清单（上线前）

首屏无报错；列表/弹层滚动穿透；键盘与底栏；iOS/Android/微信；`pnpm build`/`npm run build`。

## 8. 其他 Skill

`vue-best-practices`、`vue-router-best-practices`、`vue-pinia-best-practices`（若本仓有）。

## 附加资料

[references/vue-vant-cheatsheet.md](references/vue-vant-cheatsheet.md)：Service Mock **唯一**代码块、自定义组件 vs Vant、拆文件对照、Vue/Router/Vant 表、搜索命令。
