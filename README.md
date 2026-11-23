# 珞珈智游（微信小程序 · JS 标准模板）

> 武汉大学「大创」项目小程序，不使用云开发，原生 JS 标准模板 + 自建后端接口。

本仓库是团队协作开发仓库，**所有人以仓库代码为唯一来源**，各自在本地用微信开发者工具独立开发、调试，通过 Pull Request 合入 `main`。

| 项目 | 值 |
| --- | --- |
| 小程序名称 | 珞珈智游 |
| AppID | `wx5e97bba488bd8c72` |
| 技术方案 | 原生小程序（JS 标准模板），不使用云开发 |
| 远端仓库 | `https://github.com/azuresky4/luojiazhiyou.git` |
| 主分支 | `main` |

---

## 1. 环境要求

| 项目 | 要求 |
| --- | --- |
| 微信开发者工具 | 稳定版，建议 `1.06.2307260` 及以上 |
| 基础库 `libVersion` | `3.18.0`（与 `project.config.json` 保持一致，勿用 `trial`） |
| Node.js | `18 LTS` 及以上（仅当项目使用 npm 依赖时需要） |
| Git | `2.30` 及以上 |

> 不要用 Beta / Nightly 版工具提交代码，编译产物与调试行为可能与稳定版不一致。
> 基础库版本统一后，组员在「详情 → 本地设置 → 调试基础库」里选择同一版本，避免「我这儿正常」的问题。

## 2. 目录结构

JS 标准模板的项目根目录本身就是小程序根目录（没有 `miniprogram/` 这一层）：

```text
.
├── pages/                      # 页面，每页一个目录，含 .js/.json/.wxml/.wxss
│   ├── index/
│   └── logs/
├── components/                 # 自定义组件
│   └── navigation-bar/
├── utils/                      # 工具函数、请求封装、环境配置
│   ├── util.js
│   ├── request.js              # 网络请求封装（待建）
│   └── config.js               # 环境与接口地址（待建）
├── app.js                      # 小程序入口
├── app.json                    # 全局配置：页面路由、tabBar、window
├── app.wxss                    # 全局样式
├── sitemap.json
├── project.config.json         # 团队公共配置：appid、libVersion、编译设置   ← 入库
├── project.private.config.json # 个人本地配置                                  ← 不入库
├── package.json                # 依赖与脚本（使用 npm 时）                      ← 入库
├── package-lock.json           # 锁版本，保证组员依赖一致                       ← 入库
├── .gitignore
└── README.md
```

## 3. 首次拉取并打开项目

```bash
git clone https://github.com/azuresky4/luojiazhiyou.git
cd luojiazhiyou
npm install          # 项目使用 npm 依赖时执行；纯模板阶段可跳过
```

然后在微信开发者工具中：

1. 打开开发者工具 → **导入项目**（不要用「打开文件」）。
2. **目录**选择克隆下来的项目根目录（含 `project.config.json` 的那一层）。
3. **AppID** 填 `wx5e97bba488bd8c72`（需先被加为项目成员，见第 8 节）。
4. 若使用了 npm 依赖：菜单 **工具 → 构建 npm**，根目录会生成 `miniprogram_npm`。
5. 点击编译，模拟器正常渲染首页即导入成功。

> 首次导入后开发者工具会自动生成 `project.private.config.json`，**这是本机个人配置，不要提交**。

## 4. 本地个人配置

- `project.config.json`：`appid`、`libVersion`、编译设置等团队公共项。**改动需在 PR 中说明并请他人确认**。
- `project.private.config.json`：编译设置、个人路径、临时覆盖项（优先级更高）。已在 `.gitignore` 中，随意改，不影响他人。
- 基础库的本地覆盖会写进 `project.private.config.json`，因此「我本地 3.18.0、别人 trial」不会互相污染，但**也不会自动同步**——统一版本靠 `project.config.json`。

## 5. 接口环境约定

小程序所有后端地址统一走 `utils/config.js`，禁止在页面里硬编码 `https://` 地址。

```js
// utils/config.js —— 入库，组员共用同一套环境定义
const ENV = {
  // 本地联调：各自起本地后端，或连测试环境
  dev:  { baseURL: 'http://127.0.0.1:8000', envName: 'dev' },
  // 测试环境：部署在测试服务器的后端
  test: { baseURL: 'https://test-api.example.com', envName: 'test' },
  // 生产环境：必须是 https + 已备案域名，且已配置为 request 合法域名
  prod: { baseURL: 'https://api.example.com', envName: 'prod' }
}

// 切换环境只需改这一行；本地联调后可临时改为 ENV.dev
const current = ENV.dev

module.exports = current
```

`utils/request.js` 统一封装 `wx.request`：从 `config.js` 取 `baseURL`，统一处理 loading、错误码、`token`。

**三套环境的域名从哪里来：**

| 环境 | 来源 |
| --- | --- |
| `dev` | 组员本地后端地址，如 `http://127.0.0.1:8000`；真机联调换成电脑局域网 IP `http://192.168.x.x:8000` |
| `test` | 后端负责人在测试服务器部署后的地址（问后端要，或看后端仓库 README / 部署文档） |
| `prod` | 微信公众平台 → 开发 → 开发设置 → **服务器域名 → request 合法域名**（生产域名，必须 https、已备案、已加白名单） |

**本地联调 `http` 或局域网 IP 会被拦截**：开发者工具 → 详情 → 本地设置 → 勾选「不校验合法域名、web-view（业务域名）、TLS 版本以及 HTTPS 证书」。该勾选只影响本机调试，**上线前必须用 https 正式域名**。

## 6. 分支与协作规范

### 分支模型

| 分支 | 用途 | 规则 |
| --- | --- | --- |
| `main` | 可运行、可发布的稳定代码 | 受保护，禁止直接 push，只能经 PR 合入 |
| `feat/xxx` | 新功能 | 从最新 `main` 拉出，例：`feat/login-page` |
| `fix/xxx` | 缺陷修复 | 例：`fix/cart-total-error` |
| `hotfix/xxx` | 线上紧急修复 | 修复后同时合回 `main` |
| `docs/xxx` | 文档、注释 | 例：`docs/readme-collab` |

### 日常工作流

```bash
# 1. 同步主干
git checkout main
git pull

# 2. 开功能分支
git checkout -b feat/login-page

# 3. 开发、自测、提交
git add .
git commit -m "feat(login): 增加手机号格式校验"

# 4. 推送并开 PR
git push -u origin feat/login-page
```

- 提交前先 `git pull --rebase origin main`，保持线性历史。
- 一个分支只做一件事，PR 控制在 **400 行以内**便于评审。
- 页面文件按人分工，尽量避免两人同时改同一个 `.wxml`。

### 提交信息规范（Conventional Commits 简版）

```text
<type>(<scope>): <subject>
```

| type | 含义 |
| --- | --- |
| `feat` | 新功能 |
| `fix` | 修复缺陷 |
| `docs` | 文档 |
| `style` | 格式（不影响逻辑） |
| `refactor` | 重构（不改变行为） |
| `perf` | 性能优化 |
| `test` | 测试 |
| `build` / `chore` | 构建、依赖、杂项 |

示例：`fix(cart): 修复合计金额未保留两位小数`

### PR 流程

1. 发起 PR，标题与提交信息格式一致。
2. 按 `.github/PULL_REQUEST_TEMPLATE.md` 填写自测清单。
3. 至少 **1 名组员 review** 通过，UI 变更需附截图。
4. 由发起人合入 `main`（建议 squash merge），合并后删除远端分支。
5. 合并后其他组员 `git checkout main && git pull` 同步。

## 7. 代码规范

- 统一 2 空格缩进、单引号，提交前按项目 ESLint 配置跑一次：

```bash
npx eslint "pages/**/*.js" "utils/**/*.js" "components/**/*.js" --fix
```

- 新增页面必须：建同名四件套（`.js/.json/.wxml/.wxss`）→ 在 `app.json` 的 `pages` 中注册 → 路由路径与目录一致。
- 接口地址不硬编码，统一走 `utils/config.js`；`wx.request` 统一走 `utils/request.js`。
- 公共逻辑抽到 `utils/`，可复用 UI 抽到 `components/`，页面只做组装。
- 不使用 `console.log` 提交到 `main`，调试信息用环境开关控制。

## 8. 权限与安全

- **项目成员**：管理员在 <https://mp.weixin.qq.com> → 管理 → 成员管理 → 项目成员 添加微信账号，未加入者无法真机预览与上传。
- **仓库协作者**：GitHub 仓库 → Settings → Collaborators 添加组员账号（或让他们 Fork + PR）。
- **密钥红线**：AppSecret、上传密钥（`private.*.key`）、`.env` 一律不入库；已泄露的密钥需立即重置。
- 本仓库当前为 **Public**，任何人不经授权即可看到代码；若后续包含敏感设计，请改为 Private。

## 9. 调试与预览

| 场景 | 做法 |
| --- | --- |
| 模拟器调试 | 直接编译，配合 Sources / Network 面板 |
| 真机预览 | 工具「预览」→ 扫码（需项目成员权限） |
| 本地接口联调 | 勾选「不校验合法域名」，`config.js` 指向 `ENV.dev` |
| 体验版 | 「上传」后在公众平台设为体验版，邀请体验成员 |
| 线上问题排查 | 打开 `vConsole` 或使用小程序后台的实时日志 |
| 不同机型/基础库 | 工具内切换调试基础库，真机覆盖 iOS + Android 各一台 |

## 10. 常见问题（FAQ）

| 现象 | 原因与处理 |
| --- | --- |
| 拉取后导入报路径/配置错误 | 目录选错，应选含 `project.config.json` 的项目根目录 |
| 提示找不到依赖包 | 未执行 `npm install`，或未点「工具 → 构建 npm」 |
| 每次 pull 都冲突 | 有人提交了 `project.private.config.json`，删除本地该文件的提交并确认已被忽略 |
| 无法真机预览 / 上传 | 未在公众平台被加为项目成员，或工具未登录对应账号 |
| 编译设置被人覆盖 | 个人设置落在 `project.private.config.json`，公共项才改 `project.config.json` |
| 页面白屏 / 找不到页面 | 新页面未在 `app.json` 注册，或路径大小写不一致 |
| 请求报「不在以下 request 合法域名列表中」 | 域名未配置或用了 http；本地联调勾选「不校验合法域名」 |
| 基础库版本不一致导致 API 报错 | 在 `project.config.json` 固定 `libVersion`，不要用 `trial` |

---

## 附：快速开始（TL;DR）

```bash
git clone https://github.com/azuresky4/luojiazhiyou.git && cd luojiazhiyou && npm install
# 微信开发者工具 → 导入项目 → 选择该目录 → 填 AppID wx5e97bba488bd8c72 → 工具/构建 npm → 编译
```
