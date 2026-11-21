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

学期编码：`年 × 2 + (Fall ? 1 : 0)`，例如 Spring 2027 = 4054，Fall 2027 = 4055。

## 计划

- [ ] Google Drive 同步（`drive.appdata` scope，纯前端，用 Google Identity Services）
- [ ] 英文界面
