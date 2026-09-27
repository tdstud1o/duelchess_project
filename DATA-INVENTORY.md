# 决斗王！(duelchess) 项目数据盘点报告

> 生成时间：2026-09-27 19:53（GMT+8）
> 范围：工作区 `C:/Users/Timmoc/WorkBuddy/duelchess` 全量 + WorkBuddy 数据
> 性质：**只读盘点，未对任何文件做移动/删除/修改**

---

## 1. 总览

| 指标 | 数值 |
|------|------|
| 工作区总体积 | **372 MB** |
| 顶层条目数 | 7（5 仓库 + docs + tmp + README） |
| 文件总数（含 node_modules） | 约 7000+ |
| 可安全回收体积（估算） | ≥ **350 MB**（主要是 node_modules + 本地状态） |

**体积分布（按目录）**

| 目录 | 体积 | 占比 | 说明 |
|------|------|------|------|
| duelchess_server | 224 MB | 60% | node_modules 占 217 MB |
| duelchess | 72 MB | 19% | node_modules 占 67 MB |
| duelchess_rules | 61 MB | 16% | node_modules 占 59 MB |
| tmp（临时） | 9.9 MB | 3% | 调试/缓存/密钥/美术 |
| duelchess_mapbuilder | 65 KB | <1% | 纯配置，无依赖 |
| docs | 20 KB | <1% | 规则文档 |
| .workbuddy | 60 KB | <1% | WorkBuddy 记忆数据 |

> 结论：**~92% 的体积来自 3 个仓库的 `node_modules`**，这是可重装的生成物，并非项目真实源码。

---

## 2. 各仓库明细

| 仓库 | 总体积 | node_modules | 构建产物 | 源码特征 |
|------|--------|--------------|----------|----------|
| duelchess | 72 MB | 67 MB | dist 1.8 MB | 1585 js / 451 ts / 155 json / 106 md / 61 png / 30 wav |
| duelchess_server | 224 MB | 217 MB | .wrangler 状态 | 836 js / 609 ts / 25 sqlite* / 214 mjs |
| duelchess_rules | 61 MB | 59 MB | 无 | 305 ts / 281 js / 73 json（规则引擎，TS 为主） |
| duelchess_mapbuilder | 65 KB | 无 | 无 | 4 json / 1 html / 1 yml / 1 md（地图双层 schema） |

- `.git` 目录在各仓库均仅 1 KB（疑似 git worktree / 浅克隆指向，对象不在此处）。
- duelchess_server 内含 **39 个 sqlite 文件（合计 5.1 MB）**，均为 `.wrangler/state` 下的 miniflare 本地开发状态，非业务数据。

---

## 3. 文件类型分布（Top 项）

- **duelchess**：js(1585) map(608) ts(451) json(155) md(106) png(61) mjs(44) wav(30)
- **duelchess_server**：js(836) ts(609) map(223) mjs(214) mts(177) md(139) json(118) **sqlite(25)**
- **duelchess_rules**：ts(305) js(281) json(73) md(51) map(35) cjs/mjs(54)
- **duelchess_mapbuilder**：json(4) yml(1) md(1) html(1)

> 注：`.map`（sourcemap）在 duelchess / rules 中已全部位于 node_modules 内（仓库源码无独立 .map）；仅 duelchess_server 源码外有 7 个 .map（820 KB）。

---

## 4. WorkBuddy 数据盘点

| 路径 | 体积 | 内容 |
|------|------|------|
| `.workbuddy/` | 60 KB | 仅含 `memory/` |
| `.workbuddy/memory/2026-09-26.md` | 32 KB | 当日工作日志 |
| `.workbuddy/memory/2026-09-27.md` | 28 KB | 当日工作日志 |

- 这是 WorkBuddy 的**合法项目记忆数据**，记录每日工作流水，建议保留。
- 顶层 README.md（1.9 KB）为项目入口说明，保留。

---

## 5. 大文件与敏感数据

**>1 MB 文件清单（排除 node_modules/.git，共 4 个）**

| 文件 | 大小 | 性质 | 建议 |
|------|------|------|------|
| duelchess_server/.wrangler/.../miniflare-wobs-trace-store/...sqlite-wal | 4.0 MB | wrangler 本地观测状态 | 可清 |
| tmp/legacy_root/碧野之战-AI试稿v1.png | 2.8 MB | 棋盘美术稿 | **保留/归库** |
| tmp/legacy_root/proto1.png | 2.4 MB | 原型图 | 保留/归库 |
| tmp/legacy_root/proto2.png | 2.0 MB | 原型图 | 保留/归库 |

**🔴 敏感数据警告（切勿提交或外泄）**

| 文件 | 类型 | 处置建议 |
|------|------|----------|
| `tmp/ci-deploy-key` | SSH 私钥 | 确认已在 .gitignore；勿入库/外传 |
| `tmp/ci-server-deploy-key` | SSH 私钥 | 同上 |
| `tmp/ci-deploy-key.pub` | SSH 公钥 | 公钥可留，但配套私钥须受保护 |

> 这三个密钥文件位于 `tmp/`（非仓库 tracked 区），但务必核对任何仓库的 `.gitignore` 已覆盖，避免误提交到 GitCode/GitHub 镜像。

---

## 6. tmp/ 临时目录分析

- **体积**：9.9 MB，**577 个文件**。
- **构成**：
  - `legacy_root/`：3 张 PNG 棋盘美术 + 1 个 SVG（碧野之战宣传图/试稿）——**有价值的美术资产，建议移入 `assets/` 或地图仓库**。
  - `*.mjs` 探针（9 个）：combat-probe / debug-board / debug-resign / online-e2e 等调试脚本。
  - `*.txt`（7 个）：deploy-log / pages-deploy / domain-switch 等日志。
  - `vt/node-compile-cache/`：Node 22 编译缓存，纯可重建。
  - `vitest.config.ts.timestamp-*.mjs`：测试运行残留缓存。
  - `ci-deploy-key*`：见上节敏感数据。

> tmp/ 整体为**调试/构建副产物 + 临时资产**，除 `legacy_root` 美术外，其余均可视为可丢弃。

---

## 7. 整理建议（按风险分级）

### 🟢 安全可清理（生成物，可重建）
| 目标 | 可回收 | 恢复方式 |
|------|--------|----------|
| 3 仓库 `node_modules/` | ~343 MB | `npm install` |
| duelchess_server `.wrangler/` | ~5 MB | 重启 miniflare 自动重建 |
| tmp/vt/node-compile-cache | 少量 | Node 自动重建 |

### 🟡 需你确认（临时/调试产物，部分可能有用）
- `tmp/*.mjs` 探针、`tmp/*.txt` 日志：确认不再需要后可删。
- `tmp/legacy_root/*` 美术：建议**移动**到正式 `assets/` 目录而非删除。

### 🔴 敏感（立即核对保护状态）
- `tmp/ci-deploy-key`、`tmp/ci-server-deploy-key`：确认未进入任何仓库版本控制。

### ⚪ 保留（项目数据，勿动）
- `.workbuddy/memory/*` —— WorkBuddy 记忆
- `docs/` —— 规则与进度文档
- 各仓库 `src` / 配置 / 地图 json

---

## 8. 下一步

本报告为**只读盘点**。若需执行上述整理（删 node_modules、清 tmp、归档美术、校验密钥忽略规则），请告知具体范围，我会先给出精确的影响清单与命令，经你确认后再动手——**绝不擅自删除或移动文件**。

潜在一键回收：仅清 `node_modules` + `.wrangler` + `tmp/vt` 即可释放约 **348 MB**（占当前 94%）。
