# @melt-js/tpl-vue3-admin-ele-vben

由 [vue-vben-admin](https://github.com/vbenjs/vue-vben-admin) 5.8 的 `apps/web-ele` 拆出的 Vue 3 + Element Plus 后台模板。

- 基础库：`@melt-js/libs-vben`（上游 `packages/*` 的非组件部分）
- 组件层：`@melt-js/comps-vben`（上游 `@core/ui-kit/*` + `common-ui` / `layouts` / `plugins`）
- 模板层：本包（页面、路由、store、api、Element Plus 适配器）

## 启动

```bash
# 仓库根目录
pnpm install
pnpm vben:dev
```

打开 http://localhost:50250。

默认账号（密码均为 `123456`）：

| 用户名 | 角色 | 首页 |
| --- | --- | --- |
| `vben` | super | `/dashboard/workspace` |
| `admin` | admin | `/dashboard/workspace` |
| `jack` | user | `/dashboard/analytics` |

开发与预览环境的 `/api` 由 `vite/mock.ts` 提供最小 mock（登录 / 用户信息 / 权限码 / 菜单 / 登出 / 刷新 token），不需要额外启动后端。

## 页面

- `#/dashboard/analytics` 分析页
- `#/dashboard/workspace` 工作台
- `#/demos/element` Element Plus 示例
- `#/demos/form` VbenForm 示例
- `#/auth/login` 登录（默认）
- `#/about`、403 / 404 / 500 / 离线页

路由使用 hash 模式，便于静态预览。

## 构建 / 预览

```bash
pnpm vben:build
pnpm vben:preview   # http://localhost:50260
```

端口按仓库全局规则登记在根目录 `ports.json`：dev `50250`，preview `50260`。

## 与上游的差异

- 依赖从 `@vben/*` / `@vben-core/*` 改为 `@melt-js/libs-vben/*`、`@melt-js/comps-vben/*`。
- 原先由 `@vben/vite-config` 注入的 PWA、压缩包、importmap、license、Nitro mock 等构建插件已精简；只保留 Vue / Vue JSX、Tailwind 4、`unplugin-element-plus`、CSS layer 与本地 mock。
- 上游 `apps/backend-mock`（Nitro）没有作为独立包引入，改为 Vite middleware 的最小 mock。
- 环境变量精简为 `VITE_APP_TITLE`、`VITE_APP_NAMESPACE`、`VITE_PORT`、`VITE_BASE`、`VITE_GLOB_API_URL`、`VITE_ROUTER_HISTORY`。
- 文案、页面和 Element Plus 适配器保持上游实现。

## 目录

```text
src/
  adapter/        # Element Plus / VbenForm / VXE Table 适配
  api/            # 登录、用户、菜单、请求客户端
  layouts/        # BasicLayout / AuthLayout
  locales/        # 应用语言包
  router/         # 静态路由、动态路由、守卫
  store/          # auth store
  views/          # dashboard / demos / _core
vite/
  css-layer.ts    # 把 element-plus CSS 包进 @layer el
  mock.ts         # /api 最小 mock
```


## 发布 / unpkg

```bash
# 构建 + 打 tarball（workspace:* 会被 pnpm pack 重写）+ 提交 npm stage
pnpm publish:stage
```

生产构建使用相对路径（`VITE_BASE=./`）和上游远程 mock（`https://mock-napi.vben.pro/api`），
发布后可直接用 unpkg 打开静态页面：

```text
https://unpkg.com/@melt-js/tpl-vue3-admin-ele-vben@<version>/dist/index.html
```

stage 只是暂存，需要执行 `npm stage approve <id>` 后才会在 unpkg 上线。


## 远程组件（comps-vben）

组件层不再随模板打包，而是通过 hel-micro 远程加载：

- 本地：先启动 `pnpm vben-comps:dev`（20090），再启动模板（50250）
- 生产：从 unpkg 的 `@melt-js/comps-vben@2026.11.8-6/hel_dist/hel-meta.json` 加载
- 模板启动时在 `main.ts` 里先 `await setupRemoteComps()`，注入 Vue / Element Plus / router / pinia / i18n
  以及 preferences / stores / locales / globalState，再动态加载 `bootstrap`
- 业务代码里的组件/方法统一从 `#/remote/comps` 代理导入，类型仍来自本地 comps-vben 源码

```bash
pnpm vben:dev          # comps 20090 + tpl 50250 一起启动
pnpm vben-comps:dev    # 只启动远程组件
pnpm vben:dev:tpl      # 只启动模板（需先启动 comps）
```
