# Vue 2 → 3 与 Vant 迁移速查

> **迁入本仓库**：任务列表强制项（组件逐项 todo、每接口 DEV Mock）见主 `SKILL.md`「**迁入本仓库：强制项**」。

## 目标依赖版本（本仓库优先）

- **本 Skill 所在仓库内**做迁移、或老页**迁入本仓库**已有 Vue3 工程时：**以本仓库根目录 `package.json` 为唯一权威**（含 `dependencies` / `devDependencies` / `engines`）。升级、对齐、排错时先 `diff` 或阅读该文件，**不要**用下文外仓摘要覆盖本仓已锁定的版本（例如视口插件包名可能与参考仓不同）。
- **不在本仓库、且需与团队参考仓对齐时**：再采用主 Skill「与 sx-integration-h5-grid 对齐的目标版本」全文清单。

**版本摘要**（以根目录 `package.json` 为准，此处仅速记）：`vue` / `vue-router` / `vant` / `pinia` / `axios` / `vite` / `typescript` / `vue-tsc` 等；PostCSS 视口插件名可能与 grid 不同，**勿覆盖本仓**。

## Mock 先行（与主 Skill「1. Mock」「迁入本仓库：强制项」对应）

| 步骤     | 建议                                                                                                                                                                                                                                                              |
| -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 契约     | 列出页面用到的列表/详情/枚举/分页字段；与旧请求响应字段对齐，避免迁 Vue3 时误判为框架问题                                                                                                                                                                         |
| 落地     | **迁入本仓库**：每个新增/改动的 `src/service` 导出函数**仅允许**按下方 **「Service 层 Mock 模板」** 代码块书写（DEV Mock + `useAxiosStore` + `axios.post`）；仅替换函数名、URL、`params` 与 `data` 内容；可选 `src/mocks/*.ts` 仅作 `data` 常量拆分，不改变该结构 |
| 任务列表 | **必须**为每条待接接口建 todo，勾「完成」前须有 Mock；禁止无 Mock 标接口已迁                                                                                                                                                                                      |
| 数据量   | 列表 ≥3–5 条、含边界（空、单条、长文案）；枚举覆盖主要分支                                                                                                                                                                                                        |
| 注释     | `// 开发环境使用模拟数据` 或 `// mock: 原接口 xxx`，便于接回真实 `axios` 后删除                                                                                                                                                                                   |
| 接回     | 真实接口就绪后：保留类型与非 DEV 分支；生产构建不走 `import.meta.env.DEV` 分支                                                                                                                                                                                    |

### Service 层 Mock 模板（唯一规范）

**以下代码块为唯一允许的 service 接口形态**：不得改用其它请求封装、不得省略 DEV 分支、不得改 `Promise.resolve` 的外壳字段名（`code` / `msg` / `data`）。实现时只改：导出函数名、`axios.post` 的路径、入参、`data` 的字段与取值。`data` 须按上表「契约」填全，禁止长期 `data: {}` 占位。

**还须满足**：列表类 `data` ≥3–5 条含边界；DEV 分支旁保留 `// 开发环境使用模拟数据` 或 `// mock: ...`；`code` 与页面判断一致（常见 `0`）。

```ts
import { useAxiosStore } from "@/stores/axios-store";

export const exampleApi = (params: any) => {
  // 开发环境使用模拟数据
  if (import.meta.env.DEV) {
    return Promise.resolve({
      code: 0,
      msg: "成功",
      data: {}, // 实现时替换：与真实接口 data 完全一致，禁止长期空对象占位
    });
  }
  const axiosStore = useAxiosStore();
  const axios = axiosStore.getInstance();
  return axios.post("/path/to/real", params);
};
```

若真实接口为 **GET**：保留上列结构不变，仅将末行改为 `axios.get('/path/to/real', { params })`（或与本项目 axios 实例对 GET 的约定一致），仍须使用同一 `useAxiosStore().getInstance()` 得到的 `axios`。

## 自定义组件 vs Vant（迁入本仓库时）

| 情况                                                   | 做法                                                                                                                                                                      |
| ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 任务列表                                               | 主 Skill「**迁入本仓库：强制项**」：旧页**每个**组件 + **本仓 `src/components` 已命中**的项均须**单独 todo**（核对 props/插槽/事件后再写页面）；未完成不得用 `van-*` 顶替 |
| 旧页使用 `search`、`scrollList`、`tabsList` 等业务封装 | **不要**默认改成 `van-search` / `van-list` / `van-tabs`；先查 `src/components`，无则**迁入旧三件套 → Vue 3 SFC**                                                          |
| 本仓已有 `ScrollList.vue` 等                           | **复用**并对接 props/事件，保持列表与分页行为一致                                                                                                                         |
| 旧实现就是薄 Vant 封装                                 | 可按主 Skill「Vant 2→4」做 API 对齐                                                                                                                                       |
| 产品明确要求改版                                       | 允许改用 Vant 拼装，须写**理由与交互差异**                                                                                                                                |

**易错启发式**：「项目装了 Vant」≠「用 Vant 替换所有自定义组件」。

## 旧壳拆文件格式 → Vue 3 SFC（兼容 community-h5app 等）

旧多页壳**不等于**标准 `.vue`，迁移时先识别再合并，避免只迁其中一文件。

| 旧形态     | 典型文件                                                             | 迁入本仓库 Vue3 时                                                                                  |
| ---------- | -------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| 页面       | `index.html` + `index.js`（`Page({})`）+ `index.less` + `index.json` | 合并为 **一个** `index.vue`；`index.json` 的导航、`usingComponents` 改为路由 `meta` + 全局/按需组件 |
| 可复用组件 | `index.html` + `index.js`（`Component({})`）+ `index.less`           | 合并为 **一个** `Xxx.vue`；三份逻辑/模板/样式全量收口，必要时 less 旁挂再 `@import`                 |
| 样式依赖   | `index.less` 顶部 `@import '@/pages/common/...'`                     | 整条 import 链上影响本页的选择器须迁入或抽公共 less，**禁止**只复制 `@import` 以下片段              |

## Vue 模板与 API

| Vue 2                         | Vue 3                                              |
| ----------------------------- | -------------------------------------------------- |
| `filter`                      | 删除，改用函数/`computed`                          |
| `Vue.set` / `this.$set`       | 删除，直接赋值                                     |
| `beforeDestroy` / `destroyed` | `beforeUnmount` / `unmounted`                      |
| `.native` 修饰符              | 删除（组件 emits 对齐即可）                        |
| `Vue.prototype.xxx`           | `app.config.globalProperties.xxx`                  |
| `new Vue({ router, store })`  | `createApp` + `app.use(router)` + `app.use(store)` |
| `functional: true` SFC        | 普通组件或 `defineAsyncComponent`                  |
| `slot` / `slot-scope`         | `v-slot` / `#default="{ x }"`                      |

## Vue Router

| Vue Router 3                      | Vue Router 4                                           |
| --------------------------------- | ------------------------------------------------------ |
| `new VueRouter({ mode: 'hash' })` | `createRouter({ history: createWebHashHistory() })`    |
| `router.onReady`                  | `router.isReady()`                                     |
| 命名视图、`children`              | 概念保留，注意 `scrollBehavior` 类型与 `next` 守卫写法 |

## Vuex（若短期保留）

- Vuex 4 + Vue 3：`createStore` 与 `app.use(store)`。
- 中长期建议迁 Pinia，减少 `mapState` 等与组合式 API 混用成本。

## Vant 2 → 4（常见；运行时 ^4.9.22）

> 具体以 [Vant 官方升级指南](https://vant-ui.github.io/vant/#/zh-CN/migrate-from-v2) 为准；以下为高频项。

| 区域               | 注意点                                                                            |
| ------------------ | --------------------------------------------------------------------------------- |
| 包名               | 使用 `vant` 主包；按官方说明处理样式入口（Vite 下 less/sass 变量方式）            |
| 组件注册           | 按需自动引入时，模板里组件名大小写与解析器一致                                    |
| `Toast` / `Dialog` | 函数式 API 导入路径、链式调用与 Vant 2 可能不同；避免在模块顶层调用导致上下文错误 |
| 样式变量           | Vant 4 主题以 CSS Variables 为主，旧 Less 覆盖需改写                              |
| 移除或改名组件     | 以官方「从 v2 升级」文档列表为准，全局搜索标签名与 import                         |
| 触摸/桌面          | 仅开发环境需要 `@vant/touch-emulator`                                             |

## 建议的仓库内搜索命令（示例）

```bash
# 过滤器
rg "\|\s*\w+" --glob "*.vue"

# 已移除 API
rg "\$set|\$delete|\$on|\$off|Vue\.filter|filters:\s*\{"

# Vant 标签（按项目实际前缀调整）
rg "<van-" --glob "*.vue"
```

## 参考链接

- https://vuejs.org/guide/migration/migration-build.html（迁移构建/兼容模式，按需）
- https://router.vuejs.org/zh/guide/migration/
- https://vant-ui.github.io/vant/
