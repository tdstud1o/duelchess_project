# P1 执行提示词 — M3 在线房间「真正上线」

> 角色：执行代理（AI Agent 或开发者）。本提示词自包含，未来会话看不到上下文，所有仓库路径、命令、绑定名均已写明。
> 目标：把已提交但「仅代码层完成」的 M3（在线房间：对弈码建房 / DO 实时对战 / 服务端仲裁 / 断线重连 / 认输再战 / 棋谱落盘）从「能跑单测」推进到「公网可玩、已验证」，并给 `duelchess_rules` 加 vitest CI 门禁。

## 前置条件（先确认，不满足则停下报告）
- 本机已 `wrangler login` 且 CF 账号正确（部署目标账号须支持 `compatibility_date = "2026-08-01"` 与 SQLite-backed DO 免费计划）。
- 已 `git pull` 各子仓最新：重点 `duelchess_server`（服务端）、`duelchess`（前端）、`duelchess_rules`（规则+Ci）。
- 当前有明显并行会话在动同一批仓库——**部署前先确认工作树无半截改动**，且本次不主动 `git push` 到镜像仓（避免撞 non-fast-forward）。

## 任务分解

### T1 创建云端资源并回填 `duelchess_server/wrangler.toml`
在 `duelchess_server` 目录执行，把返回的 id 填回 `wrangler.toml` 对应占位：
```bash
cd duelchess_server
npx wrangler kv namespace create SESSIONS          # 复制 id -> [[kv_namespaces]] binding="SESSIONS" 的 id
npx wrangler kv namespace create MAP_CACHE         # 复制 id -> binding="MAP_CACHE" 的 id
npx wrangler d1 create duelchess                   # 复制 database_id -> [[d1_databases]] database_id
npx wrangler r2 bucket create duelchess-replays    # bucket 名已是 duelchess-replays，无需改
```
回填位置（`wrangler.toml`）：`SESSIONS.id`←`<KV_SESSIONS_ID>`、`MAP_CACHE.id`←`<KV_MAP_CACHE_ID>`、`DB.database_id`←`<D1_DATABASE_ID>`。DO 绑定 `ROOM`（class `Room`）+ 迁移 `v1 new_sqlite_classes=["Room"]` 已就位，不用动。

### T2 设置密钥（secret）
```bash
npx wrangler secret put ADMIN_TOKEN      # 必填建议：输入一个强随机串，mapbuilder 上传地图（P2 联调）要用
# npx wrangler secret put TURNSTILE_SECRET  # 可选：不配则 auth 跳过人机校验（auth.ts:36）
```
注意：`wrangler secret put` 需逐条交互输入值；不要在命令里明文带 token。

### T3 应用 D1 迁移并部署服务端
```bash
cd duelchess_server
npx wrangler d1 migrations apply duelchess --remote   # 执行 migrations/0001_init.sql，建 results / maps 表
npm run typecheck                                    # 部署前先过类型检查
npm run deploy                                       # = wrangler deploy
```
验收：部署后 `curl https://<your-subdomain>.workers.dev/api/maps` 返回内置地图列表（HTTP 200 JSON）。

### T4 重部署前端（Pages）并配置生产服务地址
```bash
cd duelchess
npm install
npm run build        # vite build -> dist/
# 通过 Cloudflare Pages 重新部署 dist/（wrangler pages deploy dist 或 Dashboard 关联仓库重部署）
```
**关键易漏项**：前端 `src/net.ts` 的 `serverBase()` 默认 `http://127.0.0.1:8787`（localStorage key `duelchess:server`）。`src/Online.tsx:23,141` 已有「服务地址」输入框会调用 `setServerBase` 并持久化——部署后必须在 Online 页把该地址改成 Worker 公网 URL（如 `https://duelchess-server.<sub>.workers.dev`），否则生产环境连不到后端。**建议改进（非阻塞）**：把默认地址改为生产 URL 或通过构建期环境变量注入，避免每次手动填。

### T5 给 `duelchess_rules` 加 vitest CI 门禁
在仓库根（或 `duelchess_rules`）新增 `.github/workflows/rules-ci.yml`，**必须从 `duelchess_rules` 目录运行**（workspace 根用 `--root` 跑 vitest 在沙箱会报 "No test suite found"）：
```yaml
name: rules-ci
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    defaults:
      run: { working-directory: duelchess_rules }
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: "20", cache: npm, cache-dependency-path: duelchess_rules/package-lock.json }
      - run: npm ci
      - run: npm test          # vitest run，当前 4 文件 74 it 应全绿
      - run: npm run typecheck
```

## 双端 E2E 冒烟清单（逐条勾选，作为验收口径）
1. **建房**：Online 页填昵称+选地图→建房→拿到 6 位对弈码（`POST /api/rooms` → `{code}`）。
2. **加入**：另一独立浏览器/隐身窗，用同服务地址+对弈码加入；双方都进入布局阶段。
3. **布局仲裁**：任一方提交非法布局被服务端拒绝（以 `duelchess_rules` 为准）。
4. **对局+仲裁**：走子经 `WebSocket /ws/:code?token=` 上报，服务端用 rules 包校验合法性；非法走子被拒。
5. **信息隐藏**：双方只见敌方「编号+位置」，等级全程隐藏（战斗后也不公开）——`docs/protocol.md` 视角过滤。
6. **断线重连**：一方刷新/断网→用同一 session token 重连恢复席位（`net.ts` `ensureSession` 复用 token）。
7. **认输/再战**：一方认输→服务端写 `results`（D1）+ 棋谱（R2）；可再战开新房间。
8. **棋谱落盘校验**：`GET /api/results` 能看到本局；`GET /api/replays/:id` 返回棋谱 JSON。
9. **地图上传（P2 前置）**：用 `ADMIN_TOKEN` 经 mapbuilder 或 `curl -H "x-admin-token: <token>" -X POST .../api/maps` 上传一张变体图，`GET /api/maps` 能列出。

## 风险与注意
- 并行会话可能改写同批仓库：部署前 `git status` 确认无半截未提交改动；本次**不**主动 push 镜像仓。
- `wrangler d1 migrations apply --remote` 是幂等的，但务必确认指向生产库（不是 `--local`）。
- 前端默认服务地址是 `127.0.0.1:8787`，**这是 P1 最易漏的一步**——不配生产 URL 则线上连不上后端。
- DO 用 SQLite 存储（免费计划要求），`wrangler.toml` 迁移已声明，勿删。

## 完成定义（DoD）
- [ ] 服务端 Worker 部署成功，公网 `GET /api/maps`、`POST /api/rooms` 可达。
- [ ] 两个独立浏览器会话完整跑完一局：建房→布局→对局→认输/再战，战绩与棋谱落盘可查。
- [ ] 断线后用同一 token 可恢复席位。
- [ ] `duelchess_rules` CI 门禁合入主仓，后续 PR 自动跑 vitest（74 it / 4 文件）+ typecheck。
- [ ] 复盘报告 `docs/PROGRESS-REVIEW-2026-09-27.md` 的 P1 项勾除，并记录实际 Worker 子域名与采取的默认地址方案。
