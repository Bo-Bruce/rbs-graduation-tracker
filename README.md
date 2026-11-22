# RBS–NB 毕业进度追踪 · RBS–NB Graduation Tracker

给 Rutgers Business School–New Brunswick 本科生用的选课和毕业进度工具：

- 自由组合 major（最多 3 个）和 concentration（最多 1 个）
- 每门课可以标“已修 / 在修”，自动提示缺哪门先修课
- 学期规划：历史学期、本学期、未来学期，支持自动排课
- 判断能否按时毕业：还需学分、每学期平均、先修链长度
- “换个组合会怎样”：比较加不同 concentration 或第二专业的代价
- 进度保存在自己的浏览器里，可以复制备份、导入

> **非官方工具。** 本项目由学生个人制作，与 Rutgers University 无关。课程要求以 Degree Navigator 和学术顾问（advisor）为准。

## 数据来源（2026 年 11 月整理）

- RBS–NB FA26 Foundations 301 Advising 课件
- myRBS 官网各 major / concentration 页面：<https://myrbs.business.rutgers.edu/undergraduate-new-brunswick>
- Rutgers New Brunswick Undergraduate Catalog 2024–2026（Finance 390 课程限制）

课程要求每年可能变化（例如 Fall 2026 改过 Business Core）。发现错误请提 issue。

## 部署

纯静态单文件（`index.html`），没有构建步骤。

**Vercel：** Add New Project → 选这个仓库 → Framework Preset 选 **Other** → Deploy。

**本地预览：**

```bash
python3 -m http.server 8000
# 打开 http://localhost:8000
```

## 从 Transcript 导入

上传 Rutgers unofficial transcript 的 PDF，或者粘贴 transcript 文字，自动识别已修 / 在修 / 已注册的课程。全部在浏览器里处理，不上传，也不保存姓名、学号和成绩。PDF 用 Mozilla pdf.js（只在上传时从 cdnjs 加载）读取文字，不用 AI。自动识别目前是 Beta 版：识别 RBS 必修课，并按 Rutgers 课程表提示可归入的 SAS Core；提示的分类要在“其他学分”里点“归入 SAS Core”确认后才生效（一门课可以归入多个 goal，学分只算一次）。

## SAS Core 建议（每学期更新一次）

“其他学分”里的课会根据 Rutgers 公开课程表（Schedule of Classes）提示“可满足 HST · SCL”之类的 core code，在“归入 SAS Core”时预先勾上。数据嵌在 `index.html` 的 `CORE_DATA` 里，用下面的命令更新（不需要登录）：

```bash
node tools/build-core-codes.mjs
```

## Google Drive 同步（可选）

纯前端：Google Identity Services token client + Drive REST，只用 `https://www.googleapis.com/auth/drive.appdata` scope（non-sensitive，不需要 Google 审核）。进度存在用户自己 Drive 的 `appDataFolder/tracker.json`，页面看不到 Drive 里的其他文件。`index.html` 顶部的 `GOOGLE_CLIENT_ID` 还是占位符时，同步区块不显示，页面和以前完全一样。

**Google Cloud 设置：**

1. [Google Cloud Console](https://console.cloud.google.com/) 新建一个 project。
2. APIs & Services → Library → 启用 **Google Drive API**。
3. OAuth consent screen（新版叫 Google Auth Platform → Branding / Audience / Data access）：
   - User type 选 **External**；填 App name、support email、developer contact。
   - Data access（Scopes）里加 `.../auth/drive.appdata`，不要加别的。
   - Audience 里点 **Publish app**，状态改成 **In production**。只用 non-sensitive scope，不需要提交审核；留在 Testing 的话只有 test users 能登录，而且 token 授权 7 天就失效。
4. Credentials → Create credentials → **OAuth client ID** → Application type 选 **Web application**。
   - **Authorized JavaScript origins**：`https://rbs-graduation-tracker.vercel.app`、本地测试用的 `http://localhost:8000`，需要的话再加 Vercel preview 网址。
   - Authorized redirect URIs 留空（token client 用 popup，不需要）。
5. 把生成的 Client ID（`….apps.googleusercontent.com`）填到 `index.html` 顶部的 `GOOGLE_CLIENT_ID`。Client ID 是公开值，可以提交到仓库；**不需要也不要用 client secret**。

**行为：** 连接时比较本地和 Drive 上的 `state.rev`，新的覆盖旧的；之后每次改动先存 localStorage，停 2 秒再上传。access token 大约 1 小时过期：遇到 401 先用 `prompt: ''` 静默重新取 token，浏览器拦截弹窗时显示“重新连接”。“断开”会撤销授权，本地进度和 Drive 文件都保留。

ScarletMail（Rutgers Google 账号）可能被学校拦截第三方应用，建议用个人 Gmail。

## 结构

所有代码都在 `index.html` 里：

| 部分 | 说明 |
|---|---|
| `RAW` | 课程目录：课号、名称、学分、先修（`&` 表示“且”，`/` 表示“或”） |
| `build()` | 根据 major / concentration / curriculum 版本生成要求列表 |
| `evaluate()` | 计算完成度、double count、还需学分、最长先修链 |
| `verdict()` | 判断能否按时毕业 |
| `autoPlan()` | 按先修顺序和每学期学分上限自动排课 |
| 存储 | 浏览器 `localStorage`；在 Claude artifact 中运行时改用 Claude 的 `db` 能力 |
| `drive*` | 可选的 Google Drive 同步（`appDataFolder`），只在 artifact 之外、填了 Client ID 时启用 |

学期编码：`年 × 2 + (Fall ? 1 : 0)`，例如 Spring 2027 = 4054，Fall 2027 = 4055。

## 计划

- [x] Google Drive 同步（`drive.appdata` scope，纯前端，用 Google Identity Services）— 等填 Client ID
- [x] 中文 / English / Español 界面（按浏览器语言自动选择：zh → 中文，es → Español，其他 → English；右上角可切换）
