# campus-life-assistant

学科实践项目 · 大学生生活小助手

基于 **uni-app（Vue 3 + Vite）** 的空模板，一套代码可编译到微信小程序 / H5 / App。当前仅含最小可运行骨架，业务页面待开发。

## 技术栈

| 项 | 版本 |
| --- | --- |
| uni-app | `3.0.0-alpha-5030820261124002`（Vue3 线，`vue3` dist-tag） |
| Vue | 3.4.21 |
| Vite | 5.2.8（uni-app 插件锁定的 peer 版本，不要自行升级） |
| CLI | `@dcloudio/vite-plugin-uni` |

> 所有 `@dcloudio/*` 包必须保持**同一构建号**，混用不同版本会编译报错。升级时统一改 `package.json` 里的 5 处版本号。

## 环境要求

- Node.js ≥ 18（实测 Node 24 正常）
- npm ≥ 9
- 微信开发者工具（仅微信小程序端需要）

## 快速开始

```bash
npm install

# H5：浏览器打开 http://localhost:5173
npm run dev:h5

# 微信小程序：产物在 dist/dev/mp-weixin
npm run dev:mp-weixin
```

微信开发者工具 → 导入项目 → 目录选 `dist/dev/mp-weixin`（生产包是 `dist/build/mp-weixin`），AppID 填自己的测试号或正式 AppID。

> 微信开发者工具导入后生成的 `project.private.config.json` 已被 `.gitignore` 忽略，不要提交。

## 常用命令

| 命令 | 说明 |
| --- | --- |
| `npm run dev:h5` | H5 开发服务器（热更新） |
| `npm run dev:mp-weixin` | 微信小程序开发编译（watch） |
| `npm run dev:app` | App 端开发编译 |
| `npm run build:h5` | H5 生产构建 |
| `npm run build:mp-weixin` | 微信小程序生产构建 |
| `npm run build:app` | App 生产构建 |

## 目录结构

```
.
├── index.html                 # H5 入口模板
├── vite.config.js             # Vite 配置（uni 插件注册）
├── package.json
├── src/
│   ├── main.js                # createSSRApp 入口
│   ├── App.vue                # 应用生命周期 + 全局样式
│   ├── pages.json             # 页面路由与窗口样式（必改）
│   ├── manifest.json          # 应用配置：AppID、各端设置（必改）
│   ├── uni.scss               # 全局 SCSS 变量，自动注入
│   ├── pages/
│   │   └── index/index.vue    # 首页（模板自带的占位页）
│   ├── static/                # 静态资源，原样拷贝不经过编译
│   ├── components/            # 自定义组件（待建）
│   ├── api/                   # 接口封装（待建）
│   ├── store/                 # 状态管理（待建）
│   └── utils/                 # 工具函数（待建）
└── dist/                      # 构建产物，已忽略不入库
```

## 加第一个页面的步骤

1. 新建 `src/pages/xxx/xxx.vue`
2. 在 `src/pages.json` 的 `pages` 数组里注册路径（第一项是启动页）
3. `npm run dev:mp-weixin` 重新编译，微信开发者工具自动刷新

## 配置要点

- `src/manifest.json` 的 `appid`：DCloud AppID，留空也能跑 H5 与小程序，申请后填入。
- `src/manifest.json` 的 `mp-weixin.appid`：填微信小程序 AppID；本地调试可留空并用开发者工具的测试号。
- **AppSecret 等密钥不要写进任何前端文件**，小程序包可被反编译，只放云函数环境变量或服务端。

## 开发规范

协作前必读 [Git 工作流与管理规范](docs/GIT-WORKFLOW.md)：分支模型、提交格式、PR 流程、发布与回滚规则。
