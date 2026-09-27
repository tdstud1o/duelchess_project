# 决斗王！(DuelChess) 项目进度复盘 & 下一步改进措施

> 复盘日期：2026-09-27 · **复核于 19:50**（当日下午五个仓落了一大批提交，前 11:58 版结论已失效，本版以实查 git/CI/源码重写）
> 数据来源：各子仓 `git log`/`git status`、`.github/workflows`、`wrangler.toml`、`net.ts`、`vitest` 实跑
> 修订史：① 11:54 更正 P2 误判（铁道/传送门已实现）；② 19:50 复核——M3 部署链路大半完成，前版「M3 仅代码完成未部署」证伪。

## 一、里程碑实况（实查为准）

| 阶段 | 状态 | 依据（实查） |
|---|---|---|
| M1 规则引擎 | ✅ 完成 | `duelchess_rules` **64 例 vitest 全绿**（3 文件）；草丛战斗按连通片判定 `649fd8d`、部署禁下水 `db2ccf3`、假基地回归 `855a806` 均已提交 |
| M2 前端同屏双人 | ✅ 完成 | 前端工作树 clean；`23ff300` 起多轮迭代（草丛 PRD 定稿、静音/回放工具条、部署禁下水） |
| M3 在线房间 | 🟡 部署基本就绪，待**前端 Pages 重部署 + 线上冒烟** | 见第二节 |
| M4 变体地图/回放 | 🟡 后端+规则就绪，**前端缺回放/战绩页** | 见第三节 |
| M5 注册/段位/观战 | ⏳ 未启动 | — |

## 二、M3 实际完成情况（对照 `docs/v1/P1-PROMPT.md`）

| P1 任务 | 状态 | 证据 |
|---|---|---|
| T1 创建云端资源 + 回填 `wrangler.toml` | ✅ | `wrangler.toml` 已是**真实 ID**：SESSIONS `a74643b5…`、MAP_CACHE `ff60a29b…`、D1 `71bbe89c…`、R2 `duelchess-replays` |
| T2 设置密钥 | ✅ 部署侧 / ⚠️ ADMIN_TOKEN 待确认 | `deploy.yml` 用 `CLOUDFLARE_API_TOKEN`/`ACCOUNT_ID`；`ADMIN_TOKEN` 是否 set 未验证（仅影响 P2 地图上传，不影响对局） |
| T3 D1 迁移 + 部署 | ✅ | `deploy.yml` 自动部署（push main 触发）；`825705a`「内置四张地图 + GitHub Actions 自动部署」；D1 表由 `migrations/0001_init.sql` 建 |
| T4 前端 Pages 重部署 + 生产地址 | 🟡 | `net.ts` 默认已改为 `https://duelchess-server.tmoc.qzz.io`（可 `VITE_SERVER_BASE` 或 Online 页覆盖）；**但前端无 CI，须手动 Pages 重部署** |
| T5 rules vitest CI 门禁 | ✅ | `rules-ci.yml` 已建（14:01），64 例测试绿 |

**结论：P1 的 T1 / T2(部署侧) / T3 / T5 已完成；仅 T4 的「前端 Pages 手动重部署」与「端到端冒烟」两项待办。** 服务端应已上线于 `https://duelchess-server.tmoc.qzz.io`（按自动部署 CI + 真实绑定 ID 判断），待线上冒烟最终确认。

## 三、M4 现状
- ✅ 规则引擎：铁道 `railSlide`、传送门 `teleport*` 全套、链式传送（`5445d9d` 移动-传送选项生成）。
- ✅ 前端渲染：铁路 `t-rail` / 传送门 `t-portal-gN`（Board.tsx）。
- ✅ 服务端：战绩/棋谱落盘 + `/api/results`、`/api/replays` 接口 + 内置四图。
- ❌ **前端无回放/战绩浏览页**：`App.tsx` 仅 `menu/layout/battle/online` 四 screen。
- 🟡 **变体地图上传联调**：mapbuilder → `POST /api/maps`（header `x-admin-token`）依赖 `ADMIN_TOKEN`，未跑通验证。

## 四、剩余缺口（按优先级）

1. **【P1 最关键】前端 Pages 重新部署**：前端无自动部署 CI，必须手动把最新 `duelchess` 部署到 Pages，否则生产默认地址指向的 Worker 虽在，但前端页面仍是旧版、连不上 M3。确认方式：Cloudflare Pages 控制台或 `wrangler pages deploy dist`。
2. **【P1 验收】线上端到端冒烟**：对 `https://duelchess-server.tmoc.qzz.io` 跑 建房→布局→对局→认输再战→棋谱落盘；顺带确认 rules 在 CI（ubuntu）无沙箱 EPERM 那条 "1 error" 噪声（真环境应干净）。
3. **【P2】前端回放/战绩浏览页**：服务端接口已就绪，补 `results`/`replay` 两个 screen 即可闭环 M4 前端侧。
4. **【P2】变体地图上传联调**：先确认 `ADMIN_TOKEN` 已 `wrangler secret put`，再用 mapbuilder 或 curl 上传一张变体图验证 `GET /api/maps` 能列出。
5. **【收口】聚合仓 dirty**：`README.md` / `docs/v1/init.md`（P0 编辑，M3 置 ✅）+ 四个子模块指针已 bump 但未提交。需一次 commit 收口（你此前明确不处理 project 仓，故仅标记，未动）。
6. **【文档】本复盘旧版已失效**：前版称「M3 仅代码完成未部署」「rules 未实现铁道/传送门」——现均证伪，本版已更正。

## 五、下一步建议
- **立即（P1 收尾）**：手动重部署前端到 Pages → 线上冒烟一局 → 确认 `ADMIN_TOKEN` 是否 set。
- **本周（P4 收口）**：补前端回放/战绩页；地图上传联调。
- **暂缓**：M5 注册/段位/观战；聚合仓提交（待你确认是否要我收口）。
