# 决斗王！(DuelChess) 项目进度复盘 & 下一步改进措施

> 复盘日期：2026-09-27 · 数据来源：各子仓 git 提交 / 源码实查 / 规则引擎单测实跑
> 说明：本文以**代码实际状态**为准，而非 README/init.md 中的里程碑标记（二者存在脱节，见风险 1）。
> 修订：P2（M4）经二次实查更正——rules 与前端均已实现铁道/传送门，**仅前端回放/战绩页缺失**。

## 一、实际里程碑状态

| 阶段 | 内容 | 文档标记 | 代码实况 | 结论 |
|---|---|---|---|---|
| M1 | 规则引擎（layout/move/combat/胜负 + 单测） | ✅ | `duelchess_rules` 32 例单测全绿（本次实跑通过）；含曼哈顿对称体系、炸弹前侧占比下调 | ✅ 完成且可验证 |
| M2 | 前端同屏双人 + 布局编辑器，Pages 上线 | ✅ | `Battle/Board/LayoutPhase` 实现；Pages 部署 | ✅ 完成 |
| M3 | 在线房间：对弈码匹配 / DO 实时对战 / 服务端仲裁 | ⏳（文档滞后） | 服务端 `c352efd M3 后端完成`（含断线重连/认输再战/棋谱落盘）；前端 `fcb5b3b 在线对战接入`（`Online.tsx`+`OnlineBattle.tsx`） | ⚠️ 代码完成，**未部署未端到端验证** |
| M4 | 战绩 / 棋谱回放 / 变体地图（铁道·传送门） | ⏳ | **rules 已实现**铁道滑行 `railSlide` 与传送门 `teleport*` 全套机制（含 `portals[]`/`railLinks[][]` 模型，mapAdapter 兼容 overlay 编码）；**前端 `Board.tsx` 已渲染**铁路(`t-rail`)/传送门(`t-portal-gN`)；服务端已落盘（R2 棋谱 + D1 战绩 + `/api/results`、`/api/replays`）。**唯一缺口：前端无回放/战绩浏览页**（`App.tsx` 仅有 menu/layout/battle/online 四个 screen）；变体地图上传链路（mapbuilder→`POST /api/maps`，依赖 `ADMIN_TOKEN`）待联调 | ⏳ 仅前端回放页 + 上传联调缺口 |
| M5 | 注册体系 / 积分段位 / 观战 | ⏳ | 未启动 | ⏳ 后续 |

## 二、关键风险 / 问题

1. **文档与代码严重脱节**：聚合仓 `README.md`、`docs/v1/init.md` 仍将 M3 标为「待开发」，但服务端 README 与提交均已标记 M3 完成。易误导排期。
2. **M3「完成」仅限代码层**：服务端未部署，KV/D1/R2 命名空间与 `ADMIN_TOKEN`/`TURNSTILE_SECRET` 均未创建，无线上端到端验证证据。
3. **未提交改动堆积**：
   - 前端 `src/Battle.tsx` + `src/theme.css` 有未提交 WIP（移除调试提示文案 + 主题微调）；
   - `duelchess_rules/tmp/` 未纳入 `.gitignore`；
   - 聚合仓根目录 `package-lock.json` 游离（未跟踪，空 stub）。
4. **变体地图上传链路待联调**：`mapbuilder` 已能画铁道/传送门、rules 与前端渲染均已就绪，但 mapbuilder 导出 JSON 是否含 `railLinks`/`portals` 并经 `POST /api/maps` 落库，尚未端到端验证（依赖 P1 的 `ADMIN_TOKEN`）。回放/战绩**浏览页**是前端唯一明确缺口。
5. **GitCode 镜像可能滞后**：聚合仓 `ahead of origin/main by 1 commit`，多个子仓有新提交未推。
6. **前端分支遗留**：`workbuddy/main-fbd7c83d`、`workbuddy/main-ffa7ba66` 两个 agent 分支仍存在；经 `git branch --merged main` 核查**均未合并**，删除会丢工作 → 暂不清理，待确认各自内容后单独处理。
7. **文档命名不一致**：PRD 列变体地图「勇往直前/山夹谷/长江行」，而 init.md/记忆记录为「狭路相逢/勇往直前/长江彼岸」——需对齐。

## 三、下一步改进措施（按优先级）

### P0 — 本周，低风险高收益（文档与卫生）
- **[P0-1] 同步里程碑文档**：更新 `README.md`、`docs/v1/init.md` 将 M3 置为 ✅；把变体地图命名与 PRD 对齐；标注 M4「rules+前端渲染+服务端落盘均已完成，缺前端回放/战绩页」。验收：两文档 M3 行显示 ✅，无「待开发」字样。
- **[P0-2] 收口未提交改动**：提交前端 WIP（`Battle.tsx`/`theme.css`）；在 `duelchess_rules` 加 `.gitignore` 忽略 `tmp/`；删除聚合仓根游离 `package-lock.json`。验收：`git status` 干净（除预期 submodule 指针）。
- **[P0-3] 镜像同步**：推送各子仓新提交使 GitCode 镜像同步（分支清理暂缓，见风险 6）。验收：远端镜像 commit 数与 GitHub 一致。

### P1 — M3 收口 + 验证（决定能否上线）— 2026-09-27 执行（Agent）
- **[P1-1] 服务端部署准备** ✅：KV `SESSIONS`(a74643b5…)/`MAP_CACHE`(ff60a29b…)、D1 `duelchess`(71bbe89c…)、R2 `duelchess-replays` 均已创建并回填 `wrangler.toml`；`ADMIN_TOKEN` 已设为 secret（值仅存本地记忆，不入库）。`0001_init.sql` 已 `--remote` 应用。**注意**：`workers.dev` 国内被墙，已为 Worker 绑定自定义域 `https://duelchess-server.tmoc.qzz.io`。
- **[P1-2] 前端 Pages 重新部署** ✅：新建 Pages 项目 `duelchess` 并部署（production），绑定自定义域 `https://play.tmoc.qzz.io`（CNAME→duelchess.pages.dev，已建）。默认服务地址已改为生产 URL（`net.ts` DEFAULT_BASE，`VITE_SERVER_BASE` 构建期可覆盖）——P1 最易漏项已从根上解决。
- **[P1-3] 端到端冒烟测试** ⏳ 待人工：沙箱出口网无法连通 CF 边缘（TLS reset），需 Timmoc 用浏览器按 P1-PROMPT 冒烟清单 1-9 双端验证。
- **[P1-4] CI 增加 rules 门禁** ✅：聚合仓 `.github/workflows/rules-ci.yml`（checkout submodules + npm ci/test/typecheck，working-directory=duelchess_rules）。

### P2 — M4（仅剩回放页 + 上传联调）
- ~~**[P2-1] rules M4 机制**~~ ✅ **已完成**（经二次实查，`index.ts` 含 `railSlide`/`teleport`/`teleportAttack`/`teleportMove`/`teleportMoveTeleport`/`teleportMoveSlide` 全套；`mapAdapter.ts` 支持 overlay 5/6/7→传送门组别、8→铁路）。无需再做。
- ~~**[P2-1b] 前端铁道/传送门渲染**~~ ✅ **已完成**（`Board.tsx` `t-rail`/`t-portal-gN` + 标记「〰」/「门」）。无需再做。
- **[P2-2] 前端回放 / 战绩页**：新增页面消费 `/api/replays/:id` 与 `/api/results`，复用 `Board` 渲染棋谱步进。验收：能看历史战绩列表并逐步回放一局。
- **[P2-3] 变体地图上传联调**：基于 P1 的 `ADMIN_TOKEN`，从 `mapbuilder` 走 `POST /api/maps` 上传一张含 `railLinks`/`portals` 的变体地图并校验入库（同时验证 mapbuilder 导出格式与服务端 `mapAdapter` 对齐）。验收：地图出现在 `/api/maps` 且前端能加载渲染铁道/传送门。

### P3 — M5（后续，待 M3/M4 稳定）
- **[P3-1] 注册体系 / 积分段位 / 观战**：M5 后再议，依赖 M3 在线对战与 M4 战绩数据沉淀。

## 四、一句话总结

**M1/M2 已扎实落地；M3 代码已写完但卡在「部署 + 验证 + 文档同步」；M4 的 rules 机制、前端渲染、服务端落盘三项均已完成，仅剩「前端回放/战绩浏览页」与「变体地图上传联调」两处缺口；M5 未启动。** 当前最高性价比动作是 P0 文档同步 + 收口改动，随后 P1 把 M3 真正交付上线。
