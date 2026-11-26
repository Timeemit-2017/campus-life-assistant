# Git 工作流与管理规范

> 项目：campus-life-assistant（大学生生活小助手 · 微信小程序）
> 适用对象：本仓库全部协作者
> 文档状态：v1.0 ｜ 维护人：项目负责人 ｜ 变更需走 PR

本文档规定本仓库的分支模型、提交格式、日常操作流程、代码评审、发布与回滚规则。**凡是与本文档冲突的个人习惯，以本文档为准**；确需调整的，提 PR 修改本文档，评审通过后生效。

---

## 0. TL;DR —— 每天就这六步

```bash
git switch main && git pull --ff-only          # 1. 同步主干
git switch -c feat/order-remind                # 2. 开短分支（当天开、当天合）
# ... 改代码，小步提交 ...
git add -p && git commit                       # 3. 一次提交只做一件事
git fetch origin && git rebase origin/main     # 4. 推送前先对齐主干
git push -u origin feat/order-remind           # 5. 推送并开 PR
# 6. 评审通过 → Squash and merge → 删除分支 → 本地切回 main 再 pull
```

三条铁律：

1. **任何人不得直接向 `main` 推送**，一律走 PR + Squash 合并。
2. **一个分支只做一件事**，存活时间不超过 3 天，超过就拆。
3. **密钥、`project.private.config.json`、`node_modules` 永不入库**（见第 8 节）。

---

## 1. 一次性环境配置

新同学加入后第一件事，把下面几步跑完。

### 1.1 身份信息

必须与 GitHub 账号一致，否则 PR 里的作者记录会变成陌生人：

```bash
git config --global user.name  "你的名字"
git config --global user.email "你的GitHub注册邮箱"
```

若某个邮箱不希望暴露在提交历史里，用 GitHub 提供的 `no-reply` 邮箱：
`12345678+username@users.noreply.github.com`。

### 1.2 换行符与中文路径（Windows 同学必做）

本项目跨 Windows / macOS 协作，`.gitattributes` 已统一为「仓库内 LF，检出时不转换」，但本地仍需显式设置，避免整文件 diff：

```bash
git config --global core.autocrlf false
git config --global core.safecrlf warn
git config --global core.quotepath false     # 中文文件名不再显示成 \344\275\240
git config --global i18n.commitEncoding utf-8
git config --global i18n.logOutputEncoding utf-8
git config --global pull.ff only             # pull 只允许快进，禁止意外 merge
```

### 1.3 拉取策略与别名（建议）

```bash
git config --global fetch.prune true         # 自动清理远端已删分支
git config --global rebase.autoStash true    # rebase 前自动暂存工作区
git config --global rerere.enabled true      # 记住冲突解决方式，重复冲突自动套用

git config --global alias.st  "status -sb"
git config --global alias.lg  "log --graph --pretty=format:'%C(yellow)%h%Creset %C(cyan)%ad%Creset %s %C(bold blue)<%an>%Creset' --date=short"
git config --global alias.sw  "switch"
git config --global alias.undo "reset --soft HEAD~1"
```

### 1.4 克隆与目录约定

```bash
git clone https://github.com/Timeemit-2017/campus-life-assistant.git
cd campus-life-assistant
```

- 仓库体积保持小巧：截图、原型图、设计稿等非代码资源放 `docs/assets/`，单文件 **≤ 2 MB**，超过的走网盘/图床并在文档里留链接。
- 禁止把整个 `node_modules`、微信开发者工具的编译产物塞进仓库。

---

## 2. 分支模型

采用 **轻量 Trunk-Based + PR** 模型：只有 `main` 是长期分支，所有人从 `main` 切短分支，通过 PR 合回 `main`。不设 `develop` 长期分支（人少、迭代快的团队维护双长期分支只会增加合并成本）。

### 2.1 长期分支

| 分支 | 用途 | 保护级别 |
| --- | --- | --- |
| `main` | 唯一可发布分支，随时可打包上传体验版 | 受保护，见 2.4 |

### 2.2 临时分支命名

格式：`<type>/<简短英文描述>`，可选追加 `-<issue号>`。全小写，单词用 `-` 连接，长度 ≤ 40 字符。

| 前缀 | 含义 | 示例 |
| --- | --- | --- |
| `feat/` | 新功能 | `feat/order-remind` |
| `fix/` | 缺陷修复 | `fix/login-callback-crash` |
| `hotfix/` | 线上紧急修复（从 tag 切出） | `hotfix/pay-timeout` |
| `docs/` | 仅文档 | `docs/git-workflow` |
| `refactor/` | 不改变行为的重构 | `refactor/api-layer` |
| `chore/` | 构建、依赖、配置 | `chore/upgrade-tdesign` |
| `test/` | 补测试 | `test/course-parser` |

**不允许**的命名：`test`、`dev`、`temp`、`我的分支`、`master2`、`a`。看到即视为不合规分支，评审会被打回。

### 2.3 分支生命周期

```bash
# 开分支：永远从最新 main 切出
git switch main && git pull --ff-only
git switch -c feat/order-remind

# 完工：PR 合并后删除远端分支（GitHub 上点 Delete branch，或命令行）
git push origin --delete feat/order-remind
git branch -d feat/order-remind        # 本地删
```

分支存活上限 **3 天**。若功能做不完，先合入「可运行的半成品」（用功能开关/隐藏入口屏蔽），不要挂着一个越拉越远的长分支。

### 2.4 分支保护规则（仓库维护人配置一次）

GitHub → Settings → Branches → Add branch protection rule，对 `main`：

- ✅ Require a pull request before merging
- ✅ Require approvals：**1**（3 人以上团队设 2）
- ✅ Dismiss stale pull request approvals when new commits are pushed
- ✅ Require conversation resolution before merging
- ✅ Require linear history（配合 Squash 合并）
- ✅ Do not allow bypassing the above settings（负责人例外可自行斟酌）
- ❌ Allow force pushes → 关闭
- ❌ Allow deletions → 关闭

同时开启：Settings → General → Pull Requests → 只勾选 **Allow squash merging**，取消 merge commit 与 rebase merging，保证 `main` 历史一条直线、一功能一提交。

---

## 3. 提交规范

采用 [Conventional Commits](https://www.conventionalcommits.org/) 的简化版。**type 用英文，描述用中文**——检索靠 type，可读性靠中文。

### 3.1 格式

```
<type>(<scope>): <简述>

<可选正文：为什么这么改，而不是改了什么>

<可选脚注：Closes #12 / BREAKING CHANGE: ...>
```

硬性约束：

- 首行 **≤ 50 个字符**（中文字符按 2 计），结尾**不加句号**。
- 首行使用**祈使句/动宾结构**：「修复登录闪退」，而不是「修复了登录闪退的问题」。
- 首行与正文之间空一行；正文每行 ≤ 72 字符宽。
- 一个提交只做一件事。混着格式化 + 改逻辑 + 升级依赖的提交，评审直接退回。

### 3.2 type 取值

| type | 说明 | 是否影响版本号 |
| --- | --- | --- |
| `feat` | 新功能 | minor |
| `fix` | 缺陷修复 | patch |
| `hotfix` | 线上紧急修复（仅 hotfix 分支用） | patch |
| `perf` | 性能优化 | patch |
| `refactor` | 重构，不改变外部行为 | — |
| `style` | 格式、空格、分号，不改逻辑 | — |
| `docs` | 文档 | — |
| `test` | 测试相关 | — |
| `build` | 构建脚本、依赖、包管理 | — |
| `ci` | CI 配置 | — |
| `chore` | 杂项、配置、不影响源码 | — |
| `revert` | 回滚某次提交 | — |

### 3.3 scope 约定

scope 取本项目模块名，保持以下固定写法，不要自创同义词：

`auth` `home` `course` `cafeteria` `library` `secondhand` `order` `profile` `api` `ui` `config` `deps` `docs`

示例：

```
feat(cafeteria): 新增食堂窗口实时排队人数
fix(auth): 修复微信授权回调重复触发
docs(git): 补充回滚与事故处理流程
build(deps): 升级 tdesign-miniprogram 到 1.9.2
```

### 3.4 好例子 vs 坏例子

```
# ✅ 好
feat(order): 支持按截止时间倒序展示待办
fix(api): 网络异常时不再抛出未捕获 Promise

# ❌ 坏（信息量为零 / 违反格式）
update
修改
fix bug
提交一下
feat: 改了一些东西，顺便格式化了整个文件，还升级了依赖
```

### 3.5 提交粒度与暂存技巧

用 `git add -p` 逐块挑选，把「顺手改的格式」和「真正的逻辑改动」分开提交：

```bash
git add -p                 # 交互式分块暂存
git diff --staged          # 提交前自查：这一版到底改了什么
git commit -v              # 带 diff 的提交界面，边看边写信息
```

提交前必查：

```bash
git status -sb             # 有没有误加的文件
git diff --staged --stat   # 改动面是否超出预期
```

---

## 4. 日常工作流（标准循环）

### 4.1 开工：同步主干

```bash
git switch main
git pull --ff-only            # 失败说明本地 main 有分叉，先查再动
git switch -c feat/xxx
```

### 4.2 开发：小步走

- 每完成一个「可描述的最小单元」就提交一次，不要攒一天。
- 中途要切去处理别的事：`git stash push -m "wip: order list"`，回来 `git stash pop`。
- 需要临时验证别人的分支：`git switch --detach origin/feat/other`，验完 `git switch main`。

### 4.3 推送前对齐主干

推送前先变基到最新 `main`，把冲突解决在本地：

```bash
git fetch origin
git rebase origin/main
```

冲突解决见第 5 节。若该分支已经推送过、rebase 后必须强推，用：

```bash
git push --force-with-lease
```

> 永远用 `--force-with-lease`，不要用 `--force`。前者在远端被别人更新过时会拒绝推送，避免覆盖他人提交。

### 4.4 提 PR

```bash
git push -u origin feat/order-remind
```

按 [PR 模板](../.github/PULL_REQUEST_TEMPLATE.md) 填写。PR 标题遵循与 commit 相同的格式：`feat(order): 支持按截止时间倒序展示待办`。

单个 PR 的合理规模：**改动 ≤ 400 行**。超过就拆成多个 PR——评审质量与 diff 大小成反比。

### 4.5 评审

- 作者自评一遍 diff（GitHub 的 Files changed 页面），把明显的疏漏清掉再请求评审。
- 指定至少 1 名评审人；评审人 **24 小时内**给出结论，来不及就明说。
- 评审意见用 `Must / Should / Nit` 分级标注，作者只被 `Must` 阻塞。
- 讨论串必须全部 resolve 才能合并。

### 4.6 合并

GitHub 上选 **Squash and merge**，squash 后的提交信息按第 3 节格式重写（GitHub 会把所有分支提交堆进正文，需要手动清理成一条干净的首行）。

合并后：

```bash
git switch main
git pull --ff-only
git branch -d feat/order-remind
```

### 4.7 每日收工

```bash
git status -sb                 # 工作区必须是干净的
git stash list                 # 有 stash 就处理掉或记进 issue
git lg -10                     # 确认本地 main 与远端一致
```

---

## 5. 冲突处理

### 5.1 原则

- **谁的改动后合入，谁负责解决冲突**（在 PR 侧 rebase）。
- 冲突不是「随便选一边」，必须理解两边意图后再合。
- 解决完必须本地跑一次：`npm run lint`（如有）+ 真机/模拟器点一遍受影响页面。

### 5.2 标准步骤

```bash
git fetch origin
git rebase origin/main

# 冲突文件会标出：
#   <<<<<<< HEAD        远端 main 的版本
#   =======
#   >>>>>>> feat/xxx    你的版本

git status                     # 看哪些文件冲突
# 手工编辑，逐个解决后：
git add <file>
git rebase --continue

# 想放弃这次 rebase、回到操作前：
git rebase --abort
```

### 5.3 特殊情况

| 场景 | 处理 |
| --- | --- |
| `package-lock.json` / `pnpm-lock.yaml` 冲突 | 不要手工编辑。选择一侧后删文件重新 `npm install` / `pnpm install` 生成并提交 |
| 双方都改了同一个函数 | 保留两边意图合并实现，不允许删掉别人的分支逻辑 |
| 冲突文件数超过 10 个 | 说明分支活得太久，`git rebase --abort`，考虑拆分支重做 |
| rebase 中途搞乱了 | `git reflog` 找到 rebase 前的提交，`git reset --hard <hash>` 回去（见 9.2） |

---

## 6. Code Review 规范

### 6.1 作者自查清单（提 PR 前逐项确认）

- [ ] 分支从最新 `main` 切出，且只含本功能的改动
- [ ] 提交信息符合第 3 节格式，无 `update`/`fix` 之类空提交
- [ ] 没有提交 `project.private.config.json`、`.env`、密钥、`node_modules`
- [ ] 没有遗留 `console.log`、调试断点、注释掉的死代码
- [ ] 小程序页面在真机（或至少模拟器）跑通，无白屏、无报错
- [ ] 涉及接口的改动已在 PR 描述里贴出请求/响应示例
- [ ] 新增的第三方依赖已说明理由与体积影响
- [ ] PR 描述写清「改了什么 / 为什么 / 怎么验证」

### 6.2 评审人关注点

优先级从高到低：

1. **正确性**：边界条件、空数据、网络失败、重复点击。
2. **数据与安全**：AppSecret 是否硬编码、云函数是否鉴权、用户输入是否校验。
3. **可维护性**：命名、重复代码、单文件是否过长（建议 ≤ 300 行）。
4. **性能**：`setData` 是否过大、列表是否虚拟化、请求是否重复。
5. **风格**：交给 lint，不占评审注意力。

### 6.3 表达方式

- 对事不对人，给出可执行建议：「这里建议用 `Promise.all` 并发两个请求，当前串行会多等 300ms」。
- 不阻塞的偏好用 `Nit:` 开头，作者可自行决定。
- 提出问题的一方负责确认解决，不要「提完就走」。

---

## 7. 版本发布与 Tag

### 7.1 版本号

遵循语义化版本 `MAJOR.MINOR.PATCH`：

- `MAJOR`：课程验收/正式对外版本级别的不兼容变更。
- `MINOR`：新增功能（`feat`）。
- `PATCH`：缺陷修复（`fix`/`hotfix`/`perf`）。

Tag 命名：`v1.2.0`，附注标签必须写变更说明。

### 7.2 发布流程

```bash
# 1. 确认 main 干净、CI 通过
git switch main && git pull --ff-only
git lg -5

# 2. 打附注标签
git tag -a v1.2.0 -m "v1.2.0 食堂模块上线
- 新增窗口实时排队人数
- 修复登录回调重复触发"

# 3. 推送标签（必须单独推送，git push 不会带 tag）
git push origin v1.2.0
```

4. **微信开发者工具 → 上传**，填版本号 `1.2.0` 与项目备注（备注写 tag 名，如 `v1.2.0`）。
5. 微信公众平台 → 版本管理 → 设为体验版 → 回归测试清单过一遍 → 提交审核 → 发布。
6. 发布后在 GitHub Releases 里基于该 tag 补一条发布说明（变更点 + 已知问题）。

> 体验版与正式版各自对应哪个 tag，必须记录在 Releases 中，避免「线上到底跑的哪版代码」说不清。

### 7.3 紧急修复（hotfix）

```bash
git fetch --tags
git switch -c hotfix/pay-timeout v1.2.0   # 从线上版本切出
# 修复、提交、验证
git push -u origin hotfix/pay-timeout      # 开 PR 合入 main
# 合并后：打新 tag v1.2.1 → 上传 → 提交审核（加急）
```

hotfix 合并后务必确认 `main` 已包含该修复，不要只发版不合主干。

---

## 8. 文件与敏感信息管理

### 8.1 必须入库

本项目是 uni-app（Vue 3 + Vite）工程，源码全部在 `src/` 下：

- 应用骨架：`src/main.js`、`src/App.vue`、`src/pages.json`、`src/manifest.json`、`src/uni.scss`
- 页面与组件：`src/pages/`、`src/components/`、`src/static/`、`src/utils/`、`src/api/`、`src/store/`
- 工程配置：`package.json`、`package-lock.json`、`vite.config.js`、`index.html`
- 协同配置：`.gitignore`、`.gitattributes`、`.editorconfig`、CI 配置
- 云函数源码 `cloudfunctions/<fn>/index.js` 与 `package.json`（**不含** `node_modules`）
- 文档

### 8.2 禁止入库

| 类型 | 举例 | 处理方式 |
| --- | --- | --- |
| 个人 IDE 配置 | `project.private.config.json`、`.hbuilderx/` | 已在 `.gitignore` 中忽略 |
| 依赖目录 | `node_modules/`、`.npm-cache/` | 忽略，靠 `npm install` 生成 |
| 密钥凭证 | AppSecret、云环境密钥、第三方 API Key、`.env` | 用 `.env.example` 占位 + 本地 `.env` |
| 构建产物 | `dist/`、`unpackage/`、`.miniprogram/` | 忽略，构建时生成 |
| 系统垃圾 | `.DS_Store`、`Thumbs.db`、`desktop.ini` | 忽略 |
| 大二进制 | 视频、PSD、整包设计稿 | 放网盘/图床，仓库留链接 |

`package-lock.json` **必须入库**——锁文件是团队依赖一致的唯一保障；`.gitattributes` 已把它标记为 `merge=binary`，冲突时删掉重装而不是手工合并。

### 8.3 密钥的正确姿势

小程序前端代码里**不放任何秘密**——小程序包可被反编译，打进 `dist/` 的 AppSecret 等于公开。

- AppSecret、云开发密钥只放**云函数环境变量**或服务端。
- 本地开发用的配置写 `.env`，模板提交 `.env.example`：

```
# .env.example
CLOUD_ENV_ID=your-env-id
API_BASE=https://example.com/api
```

- 需要前端读取的环境差异（体验版/正式版域名）走 `wx.getAccountInfoSync().miniProgram.envVersion` 判断，而不是塞密钥。

### 8.4 已误提交密钥怎么办

**删文件再提交一次是不够的**——历史里仍在。按顺序执行：

1. 立即到对应平台**吊销并轮换**该密钥（这是唯一真正的补救）。
2. 通知负责人，仓库需做历史清理（`git filter-repo` 或 BFG），由负责人统一操作并让全员重新克隆。
3. 如果仓库是公开的，视为已泄露，按事故记录（第 10 节）。

---

## 9. 事故处理与回滚

### 9.1 已合并到 main 的错误提交（推荐 revert）

不改写历史，安全可追溯：

```bash
git switch main && git pull --ff-only
git revert <bad-commit-hash>          # 生成一条反向提交
git push origin main                  # 若 main 受保护，则走 PR
```

回滚一个 squash 合并的 PR，直接 revert 那个 squash 提交即可。

### 9.2 本地搞乱了（reflog 救命）

```bash
git reflog                            # 列出所有 HEAD 移动记录
git reset --hard <hash>               # 回到指定状态（丢弃工作区改动）
git reset --soft <hash>               # 回到指定状态（改动留在暂存区）
```

`git reflog` 是本地的「操作录像」，rebase 错了、reset 错了、分支误删了，**先别慌，先看 reflog**，90 天内都能找回。

### 9.3 只想拿别人的某一次提交

```bash
git cherry-pick <hash>                # 把指定提交复制到当前分支
```

用于「hotfix 修好了，但 main 上还没这个修复」之类的场景。注意 cherry-pick 会产生新的 hash，后续两边都改这块时容易冲突。

### 9.4 误删了未合并的分支

```bash
git reflog | Select-String "feat/xxx"     # PowerShell
git reflog | grep "feat/xxx"              # bash
git switch -c feat/xxx <hash>
```

### 9.5 事故记录

发生「主干被污染 / 密钥泄露 / 线上回滚」时，在 `docs/incidents/` 下新建 `YYYY-MM-DD-简述.md`，写清：现象、影响范围、时间线、根因、补救措施、后续预防。**不追责，只追流程漏洞。**

---

## 10. 常见坑清单

| 现象 | 原因 | 处理 |
| --- | --- | --- |
| 整个文件都显示改动，但只改了一行 | 换行符 CRLF/LF 混用 | 确认 `.gitattributes` 生效，`git add --renormalize .` 后提交一次 |
| 中文文件名显示成 `\344\275\240` | 未设 `core.quotepath` | `git config --global core.quotepath false` |
| `git pull` 莫名产生 merge 提交 | 未设 `pull.ff only` | 见 1.2，并 `git config --global pull.ff only` |
| 提交信息中文乱码 | 终端编码问题 | 1.2 里的 `i18n.*Encoding` 两条 |
| PR 显示 200 个文件改动 | 分支太久没同步 + 顺便格式化了 | 拆分支，格式化单独一个 `style:` 提交 |
| 合并后 `main` 跑不起来 | squash 时丢了依赖锁文件或漏提交新文件 | 立刻 `git revert`，重新提 PR 修 |
| `--force` 覆盖了同事的提交 | 用了 `--force` | 用 `--force-with-lease`；被覆盖的提交用 reflog 在同事机器或远端恢复 |
| 小程序包体积超限（主包 > 2 MB） | 大图/依赖直接进包 | 图片走 CDN，检查 `dist/build/mp-weixin/common/vendor.js` 体积，拆分 `subPackages` |

---

## 11. 命令速查

```bash
# —— 状态与历史 ——
git status -sb                          # 简洁状态
git lg -20                              # 图形化历史（需 1.3 的别名）
git diff                                # 未暂存改动
git diff --staged                       # 已暂存改动
git log -p --follow -- path/to/file     # 某文件的完整变更史

# —— 日常 ——
git switch main && git pull --ff-only   # 回主干并同步
git switch -c feat/xxx                  # 开分支
git add -p                              # 分块暂存
git commit -v                           # 带 diff 提交
git stash push -m "wip"                 # 暂存现场
git stash list / pop / drop             # 管理 stash

# —— 同步与整理 ——
git fetch origin && git rebase origin/main
git push -u origin feat/xxx
git push --force-with-lease             # rebase 后强推（唯一允许的强推形式）
git rebase -i HEAD~3                    # 整理最近 3 个提交

# —— 撤销 ——
git restore <file>                      # 丢弃工作区改动
git restore --staged <file>             # 取消暂存
git commit --amend                      # 修改最后一次提交（未推送时）
git revert <hash>                       # 安全回滚已推送提交
git reset --hard <hash>                 # 本地回退（慎用）
git reflog                              # 一切误操作的后悔药

# —— 发布 ——
git tag -a v1.2.0 -m "说明"
git push origin v1.2.0
```

---

## 12. 附录

### 12.1 提交信息模板（可选启用）

```bash
git config --global commit.template .gitmessage
```

`.gitmessage` 内容见仓库根目录（启用后 `git commit` 会带出注释提示，`#` 开头行不入库）。

### 12.2 提交信息校验钩子（可选）

在 `.git/hooks/commit-msg` 放置脚本可强制首行格式（注意：`.git/hooks` 不入库，需每人自行安装，脚本见 `docs/hooks/commit-msg`）。不强制，但强烈建议在有 CI 之后改用 CI 校验，避免依赖本地钩子。

### 12.3 本文档的修改方式

1. `git switch -c docs/git-workflow-<变更点>`
2. 修改 `docs/GIT-WORKFLOW.md`
3. 提 PR，标题 `docs(git): <变更摘要>`，说明「为什么要改这条规则」
4. 至少 1 名评审人同意后合并

### 12.4 相关文件

- [`.gitignore`](../.gitignore) —— 忽略规则
- [`.gitattributes`](../.gitattributes) —— 换行符与文本属性
- [`.github/PULL_REQUEST_TEMPLATE.md`](../.github/PULL_REQUEST_TEMPLATE.md) —— PR 模板
- [`.gitmessage`](../.gitmessage) —— 提交信息模板
