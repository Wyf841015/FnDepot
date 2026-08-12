# 清理精灵 FnClearup

> 智能扫描系统，精准识别已卸载应用、网盘挂载和 Docker 的残留目录及重复文件，一键安全清理 - Node.js 版。

![清理精灵 FnClearup](ICON_256.png)

## v0.9.5 更新

- **修复：去重浏览弹窗 ENOENT 报错** — `showDupBrowseModal` 默认从 `DupState.paths` 最后一个路径开始浏览，当路径不存在（如 `/vol1/1000/music`）时 API 返回 500 ENOENT，弹窗显示错误。改为从 `/` 根目录开始浏览
- **修复：根路径卷条目不可点击** — 根路径 `/` 浏览返回的卷条目缺少 `isDirectory:true`，前端渲染为不可点击的 📄 文件图标，导致只能添加 vol1 下目录无法添加 vol2/vol3
- **测试覆盖：87**

## v0.9.2 更新

- **新增：去重多目录扫描** — 目录浏览支持选择多个目录同时查重，音乐/文件去重面板均可添加多个扫描路径
- **修复：去重扫描后端** — `handleDupScan` 接受 `paths` 数组替代单 `path`，遍历所有路径收集文件
- **修复：外部分支扫描脚本** — `dup_scan.js` 支持逗号分隔多路径参数
- **测试覆盖：84**

## v0.9.1 更新

- **修复：P0 文件名截断损坏** — `app.js` / `empty_dir.js` / `temp.js` 三处 trash 重命名用 `slice(0, ext.length)` 保留 stem 前 N 字符（例：`data.bin.gz` → `dat_1.gz`），改为 `slice(0, -ext.length)` 正确去掉扩展名
  - `app.js` 删 29 行重复 `moveToTrash`，统一 import from `trash.js`
- **修复：execCmd 超时不杀进程** — `spawn({timeout})` 超时后只发 SIGTERM 不强杀，`rm -rf` 在 fuse 挂载点挂死后事件循环永久阻塞。改为 SIGTERM → 5s 宽限期 → SIGKILL 兜底
- **修复：bigfiles sibling threshold 测试假阳性** — 原测试永远不触发阈值（fixture 只有 1 文件），断言 `typeof === 'boolean'` 空通过。新增独立测试 `tests/test_bigfiles_sibling_threshold.js`：环境变量 `TRM_FNCLEARUP_SIBLING_THRESHOLD=5` + 7 entry fixture + 断言 `skipped_subtree===true`
- **测试覆盖：84 → 85**（含 standalone sibling threshold 测试）

## v0.9.0 更新

- **🐳 新增：Docker 清理 Tab** — 一键清理已停止容器 (`exited`/`created`/`dead` 状态) 和 Docker Build Cache
  - 容器 ID 严格 regex 校验（`^[a-f0-9]{12,64}$`），防止参数注入
  - `dry_run: true` 默认强制预览，删除前必显式确认
  - `/var/run/docker.sock` 不存在 → 干净降级到「Docker 未检测到」独立 panel
- **🐘 新增：大文件查找器 Tab** — 跨 `/vol*` 卷扫描 ≥ 100 MB 文件 Top 100，支持 minSize/topN/depth 三档过滤，5min hard timeout + setImmediate yield 可在 ~1s 内停止
- **🧹 修复：sysclean HOME 硬编码**（P0 真 bug）— `root` 跑只清 root 缓存，多用户 NAS 漏清理其他用户的 npm/pip/uv/Chromium 等
  - `getUserHomes()` 自动发现所有 `/vol<N>/<UID>` 数字 UID 目录
  - USER_CACHE_TEMPLATES × N 用户展开成独立清理项
  - `excludeSingleChildNames` 防重复
- **测试覆盖：60 → 78**（+18 新增，含恶意 ID 注入防护测试）

## v0.8.1 更新

- **修复：定时清理全面代码审计** — 审查发现并修复 8 项 P1 问题，涵盖 API 字段不匹配、路径提取错误、HTML 属性注入等

  - **应用残余不清理** — `/api/scan` 返回 `orphan` 对象数组未取 `.path`，全被 `isSafePath` 过滤
  - **Docker 清理不工作** — `imgRes.data?.unused` 路径错误（应为 `?.images?.unused`），`volRes.data?.orphans` 路径错误（应为 `?.volumes?.orphan`），`netRes.data?.networks` 是对象不是数组 → 崩溃
  - **Temp 清理不工作** — 用 `f.atime` 过滤但 API 返 `age_hours`，`delRes.data?.deleted` 是数组当数字
  - **Trash 清理不工作** — `data.totalItems/size` 不存在（实际在 `data.totals`），`cleanRes.data?.cleaned` 不存在（实际是 `deleted.length`）
  - **镜像删除计数为 0** — 读 `deleted` 但 API 返 `total`
  - **磁盘清理死代码** — 读 `orphans` 但 API 返 `vol02_dirs`
  - **🧾 文件清单按钮无反应** — `JSON.stringify` 双引号闭合 HTML `onclick` 属性
  - **3 处非原子写入** — `writeFileSync` 直接写，崩溃丢数据
  - **mkdirSync 无异常保护** — 磁盘满时崩溃
  - **isRunning 锁无 finally** — 异常时永久锁死定时器

- **新增：定时清理计划** — 支持 5 种清理类型（应用残余/网盘挂载/Docker/tmp/回收站），可配置时间、间隔，自动执行并生成清理报告
- **新增：定时清理前端配置** — 独立设置面板，各清理类型可独立启停，支持子选项（Docker 镜像/卷/网络分别控制）
- **新增：定时清理执行报告** — 每次清理自动生成详细报告（清理项/错误/耗时），保留历史记录与报告文件
- **密码和配置的0.7.x遗留问题修复** — 代码审计 14 项 P0/P1/P2 全面修复

## v0.7.12 更新

- **新增：去重结果前端分页** — 大量重复文件时分批加载，新增「加载更多」按钮
- **优化：去重 KPI 卡片显示总数** — 后端返回总匹配组数，前端实时展示

## v0.7.11 更新

- **修复：系统清理忽略 `/boot` 目录** — 移除 SCAN_TARGETS 中的 `old-kernels` 项（vmlinuz/initrd.img/System.map 等关键内核文件），避免误删导致系统无法启动。内核清理应通过 `apt` 单独处理

## v0.7.10 更新

- **新增「🗑 回收站」Tab** — 批量清理 `/vol<N>/<UID>/.@#local/trash/`，按 mtime 30/90/365 天分级清理
- **KPI 五联** — 30 天+ / 90 天+ / 1 年+ / 总占用 / 顶级目录数（颜色梯度从浅到深绿）
- **5 档清理按钮** — `30 天+` / `90 天+` / `1 年+` / `🔥 全部清空`（二次确认）/ `👁 浏览内容`（只读）
- **多 vol × 多 UID 扫描** — 自动扫 `/vol1-10` + `/vol01-09` + `/vol02`（fnOS 网盘 fuse mount）× 所有数字 UID
- **路径遍历防护** — `isTrashPath` 严格白名单 + `..` 段直接拒
- **Manifest 备份** — 清理前自动写 JSON 到 `data/manifests/`，记录 path/size/mtime/note「回收站删除后无法恢复」
- **root 权限包含所有用户** — fnclearup 以 root 跑，扫描全设备 trash 目录，单表展示（vol + UID + 路径 + 文件数 + 目录数 + 大小 + 最老 mtime）
- **回收站清理预览按 mtime 过滤** — 弹窗只列实际会被删的目录（之前 bug：列了全部 4 个 trash 目录不管 mtime）
- **回收站 Tab 移动端自适应** — 5 个 KPI 保持全局 3 列布局（跟其他面板一致），移动端 480px 下按钮变 2 列网格 + 表格切卡片布局 + 2 个 modal 全屏
- **新增「🧹 系统清理」Tab** — 17 项扫描目标：APT 缓存/列表、syslog .gz/.1、journal、npm/pip/uv/node-gyp/typescript 缓存、4 个浏览器缓存、Playwright、其它 ~/.cache、应用日志 (>50MB)、旧内核（危险默认不勾）
- **系统清理风险过滤** — 全部 / 仅低 / 中+低 三档 radio，KPI 实时显示可清理项 / 总可释放 / 已选中
- **「✅ 推荐清理」一键按钮** — 自动勾选 10 项 100% 无风险的（apt / syslog / npm / pip / uv / node-gyp / typescript），其他项系统自动重建
- **清理前自动备份 manifest** — 开关默认 ON（localStorage 持久化），清理前写 JSON 到 `data/manifests/`
- **「📜 历史清单」按钮** — 弹窗列出所有历史 manifest（mtime 倒序），支持「👁 查看」+「⬇ 下载」

## v0.7.9 更新

- **修复：扫描期间点击其他功能（如目录浏览）无响应** — 根因是空目录扫描的 `walk()` 是同步递归调用，会**阻塞整个 Node.js 事件循环**。扫描运行时整个 server 停止响应所有 HTTP，浏览/stop 请求排队等扫描完才被处理
- **修复：walk/collect 改为 async，每 200 目录 `await setImmediate()` 让出事件循环** — 这样扫描期间 server 仍能响应 browse/stop
- **修复：emptyDirJobs 在 setTimeout 前先注册** — 这样 stop 请求**立即**能找到 job（之前要等 runScan 整个 promise 链完成后才注册）
- **新增 stop 提前到达的处理** — 用 `aborted_flag.early` 标记 stop 比 scan 早到的情况，runScan 解析后检查并立即退出
- **测试 60/60 通过** — 新增用例：扫描 500 目录中途并发发 stop + browse，验证两者都在 5s 内响应

## v0.7.8 更新

- **新增启动警告弹窗** — 每次打开应用立即弹出「⚠️ 清理有风险，确认需谨慎！！！」，必须阅读 3 秒后才能点「✅ 我已知晓风险」（防止用户没读就点）
- **弹窗内容** — 红底警示框列出 3 条强制确认项（已备份/了解目录含义/勾选前再次确认）+ 强推荐「清理前写 manifest」开关提示 + 回收站恢复路径说明
- **关闭行为** — 「我已知晓风险」关闭弹窗 + toast 提示；「稍后再说」也关闭弹窗（每次重新进入应用都会再弹）

## v0.7.7 更新

- **Manifest 目录调整** — 改写到 `<TRM_PKGVAR>/data/manifests/`，跟 `info.log`、`app.pid` 在同一个父目录（fnOS 标准 `data/` 子目录组织）。不再写到 `/vol*/UID/.@#local/`
- **删除 UI 提示** — 开关旁"写到 .@#local/，仅记路径不复制文件"那句删了，确认弹窗和 toast 文案同步更新
- **`getSafeTrashDir` → `getManifestDir` 重命名** — 旧函数保留为 alias，避免破坏其他引用
- **测试 58/58 通过** — 新增 2 个用例验证 `base_dir` 和 `backup_path` 都在 `/data/manifests/` 下

## v0.7.6 更新

- **🧹 新增「系统缓存清理」Tab** — 14 项可清理：APT 缓存/列表、系统日志轮转、NPM/PIP/UV/Node-gyp/TypeScript 缓存、浏览器缓存、Playwright 浏览器、其它 `~/.cache` 自动发现、应用日志、旧内核
- **✅ 「推荐清理」一键勾选** — 10 项低风险、auto-rebuild 的安全项（apt/npm/pip/uv/.gz 等），最常用的"无感清理"
- **📦 清理前备份 manifest** — 每次清理前先写一份元数据（路径/size/mtime/时间戳/工具版本/主机/用户）到 `<TRM_PKGVAR>/data/manifests/fnclearup_pre_clean_manifest_<时间>.json`（仅记录，不复制文件）。跟 `info.log`、`app.pid` 在同一个父目录（`<TRM_PKGVAR>`），按 fnOS 标准 `data/` 子目录组织
- **📜 历史清单弹窗** — 「📜 历史清单」按钮列出所有历史 manifest，可下载查看（审计/回溯用）
- **新 API 端点** — `GET /api/sysclean/manifests` 列出、`GET /api/sysclean/manifest?path=...` 下载
- **安全加固** — `handleSysCleanGetManifest` 白名单（必须位于 `getSafeTrashDir()` 下 + 文件名 `fnclearup_pre_clean_manifest_` 前缀 + `.json` 后缀）拒绝路径遍历
- **风险过滤 UI** — 「全部 / 仅低风险 / 中+低」radio 切换即时刷新列表
- **getSafeTrashDir() 工具函数** — 按优先级返回 `getSafeTrashDir()`：用户 vol1-vol10 → `/var/fnclearup` → 测试模式 `/tmp/fnclearup_test/manifests`
- **测试 56/56 通过** — 新增 6 个用例：备份写入、不备份、清单列表、清单下载、路径遍历拒、缺参拒

## v0.7.5 更新

- **5 个 modal「确认删除」按钮修复** — 之前调错函数（`confirmXxx` 而非 `doXxx`），点击只重开 modal 不执行删除
- **安全加固 (P0/P1)** — 4 个删除端点加 `isSafePath` 白名单，handleBrowse 加 `/vol*` 白名单，dup export 加 CSV 注入防护
- **删除用户强警告** — 双重 `confirm()` 弹窗 + ⚠️ 红色警告
- **Docker 删除确认** — 3 个 docker 删除（卷/网络/镜像）加 modal-overlay 确认弹窗
- **empty_dir 重写** — 加 stop API、跳过 `@app*` 系统目录、流式扫描 + state file 保留（修复 complete 状态提前消失 bug）
- **数据隐私清理** — 删除 `app/data/` 4 个测试 JSON（含真实扫描路径）
- **CORS 环境变量** — 改用 `CORS_ORIGINS` 逗号分隔
- **删除冗余代码** — `app/ui/utils.js` 3 个未使用函数清理
- **测试 39/39 通过** — 新增路径遍历、CSV 注入、HTML onclick 契约测试

## v0.7.2 更新

- **空目录清理** — 新增「去空」选项卡，扫描目录下的空目录，一键清理（单删/批量删/回收站/永久删）
- **实时扫描进度** — 扫描中实时显示当前正在扫描的目录路径，每 10 个目录更新一次
- **移动端适配** — 空目录列表支持水平滚动和路径自动换行
- **错误防护升级** — 加入 `uncaughtException` 和进程退出码检测，子进程崩溃不再静默消失
- **忽略 .@#local** — 空目录扫描跳过 fnOS 回收站目录
- **版本号统一为 0.7.2**

## 功能特性

- **自动扫描** — 动态发现系统中所有 vol 卷（`/vol1` ~ `/vol10` 及 `@app*` 目录）
- **孤立目录检测** — 交叉比对已安装应用列表与实际文件目录，精准识别残留孤立目录
- **多选删除** — 支持批量勾选，一键清理，附确认弹窗与路径预览
- **明暗主题** — 支持手动切换（深色/浅色）
- **Tab 切换** — 应用 / 网盘 / Docker / tmp / 去重 / 去空 / 系统清理 / 回收站 独立展示
- **KPI 卡片** — 显示卷总数、已挂载数量、未挂载数量
- **Docker 卷管理** — 查看在用卷/残余卷，一键批量删除
- **Docker 网络管理** — 扫描未使用网络，一键批量删除
- **Docker 镜像管理** — 查看未使用镜像，一键批量删除
- **tmp 清理** — 扫描 `/tmp` 和 `/var/tmp` 下 24h+ 未访问的文件，一键清理
- **文件去重** — SHA-256 哈希比对，智能分组显示重复文件
- **音乐去重** — 基于元数据（ID3/FLAC）的智能重复音乐识别
- **去重多线程哈希** — 10 线程并行计算文件哈希，动态调度
- **空目录清理** — 扫描指定目录下的空目录（全空目录合并显示），支持回收站/永久删除
- **目录浏览选择** — 内置文件选择器，支持浏览并选择扫描目录
- **系统清理** — 17 项扫描目标（apt / syslog / journal / 包管理器缓存 / 浏览器 / Playwright / 旧内核），三档风险过滤 + 一键推荐清理 + 清理前 manifest 备份
- **回收站批量清理** — 扫描所有 vol × 所有 UID 的 `.@#local/trash/`，按 mtime 30/90/365 天分级清理，root 权限包含全部用户
- **启动警告弹窗** — 每次进入应用立即弹「⚠️ 清理有风险」红底警示（3 秒倒计时）
- **响应式布局** — 适配桌面端与移动端
- **轻量架构** — 纯 Node.js ESM 实现，零 npm 依赖

## 技术架构

```
清理精灵 FnClearup (Node.js)
├── app/ui/                # 前端界面（HTML/CSS/JS）
│   ├── index.html        # 主页面
│   ├── main.js           # 前端逻辑
│   ├── server.js         # HTTP + UNIX Socket 服务器
│   ├── api/              # API 路由处理器
│   │   ├── app.js        # 应用扫描/删除
│   │   ├── disk.js       # 网盘扫描
│   │   ├── docker.js     # Docker 资源管理
│   │   ├── temp.js       # 临时文件清理
│   │   ├── dup.js        # 去重引擎
│   │   ├── empty_dir.js  # 空目录扫描/删除
│   │   ├── sysclean.js   # 系统清理（apt/syslog/缓存/旧内核）
│   │   └── trash.js      # 回收站批量清理
│   ├── lib/              # 通用工具
│   │   ├── config.js     # 全局常量（版本、目录等）
│   │   └── util.js       # 工具函数
│   ├── styles/           # 样式文件
│   └── images/           # 图片资源
├── app/empty_dir_scan.js # 空目录扫描（独立运行用）
├── app/dup_scan.js       # 去重扫描子进程
├── app/dup_hash_pool.js  # 10线程哈希计算池
├── app/dup_hash_worker.js# 哈希计算工作线程
├── data/manifests/       # 清理前备份（PKGVAR 下）
├── cmd/                  # FnOS 生命周期脚本
├── config/               # 权限配置
└── wizard/               # 安装向导
```

- **后端**：Node.js ESM（零 npm 依赖，仅用内置模块）
- **前端**：原生 HTML/CSS/JS，无框架依赖
- **通信**：HTTP API 请求 + UNIX Socket（fnOS Gateway 转发）
- **数据存储**：JSON 文件状态 + 去重结果持久化
- **fnOS Gateway**：自动挂载 `/app/fnclearup` 前缀
- **依赖**：需 Node.js v24 运行时

## API 端点

| 端点 | 方法 | 描述 |
|------|------|------|
| `/api/version` | GET | 获取版本号 |
| `/api/ping` | GET/POST | 健康检查 |
| `/api/scan` | POST | 扫描孤立目录 |
| `/api/delete` | POST | 删除目录/用户 |
| `/api/mounts` | GET | 获取网盘挂载列表 |
| `/api/vol02` | GET | 获取 vol02 目录 |
| `/api/volumes` | GET | 获取 Docker 卷 |
| `/api/volumes/delete` | POST | 删除 Docker 卷 |
| `/api/networks` | GET | 扫描 Docker 网络 |
| `/api/networks/delete` | POST | 删除 Docker 网络 |
| `/api/images` | GET | 获取 Docker 镜像 |
| `/api/images/delete` | POST | 删除 Docker 镜像 |
| `/api/system` | GET | 系统资源概览 |
| `/api/temp/scan` | GET | 扫描临时文件 |
| `/api/temp/delete` | POST | 删除临时文件 |
| `/api/dup/scan` | POST | 开始去重扫描 |
| `/api/dup/status` | GET | 获取去重扫描状态 |
| `/api/dup/delete` | POST | 删除去重结果 |
| `/api/dup/stop` | POST | 停止去重扫描 |
| `/api/dup/pause` | POST | 暂停去重扫描 |
| `/api/dup/resume` | POST | 继续去重扫描 |
| `/api/dup/export` | POST | 导出去重结果 |
| `/api/jobs` | GET | 获取去重任务列表 |
| `/api/browse` | GET | 目录浏览（参数：path） |
| `/api/empty_dir/scan` | POST | 开始空目录扫描 |
| `/api/empty_dir/status` | GET | 获取空目录扫描状态 |
| `/api/empty_dir/delete` | POST | 删除空目录 |
| `/api/sysclean/scan` | GET | 扫描系统清理目标（17 项） |
| `/api/sysclean/clean` | POST | 清理选中的项（写 manifest + 删除） |
| `/api/sysclean/manifests` | GET | 列出历史 manifest |
| `/api/sysclean/manifest` | GET | 下载/查看单个 manifest |
| `/api/trash/scan` | GET | 扫描所有 vol × UID 的 trash 目录 |
| `/api/trash/browse` | GET | 浏览某 trash 目录的文件树（只读） |
| `/api/trash/clean` | POST | 清理 trash（按 mtime 阈值 + manifest 备份） |

## 版本历史

### v0.9.5 (2026-08-12)

- 修复：去重浏览弹窗默认路径不存在时报 ENOENT
- 修复：根路径卷条目缺少 isDirectory:true 不可点击
- 测试覆盖：87

### v0.8.0 (2026-06-15)

- 新增：定时清理计划 — 支持 5 种清理类型（应用残余/网盘挂载/Docker/tmp/回收站），可配置时间、间隔，自动执行并生成清理报告
- 新增：定时清理前端配置面板，各清理类型独立启停
- 新增：定时清理执行报告（清理项/错误/耗时），保留历史记录
- 修复：代码审计 14 项 P0/P1/P2 全面修复

### v0.7.12 (2026-06-08)

- 新增：去重结果前端分页渲染 + 加载更多
- 优化：去重 KPI 卡片显示总数

### v0.7.11 (2026-06-04)

- 修复：系统清理忽略 `/boot` 目录 — 移除 SCAN_TARGETS 中的 `old-kernels` 项，避免误删内核文件

### v0.7.10 (2026-06-03)

- 新增「🗑 回收站」Tab — 批量清理 `/vol<N>/<UID>/.@#local/trash/`，按 mtime 30/90/365 天分级清理，root 权限包含全部用户
- 新增「🧹 系统清理」Tab — 17 项扫描目标（apt/syslog/journal/包管理器缓存/浏览器/Playwright/旧内核），三档风险过滤 + 一键推荐清理 + 清理前 manifest 备份
- 回收站清理预览按 mtime 阈值过滤（之前 bug：列了全部 trash 目录不管 mtime）
- 回收站 Tab 移动端自适应（480px：按钮 2 列 + 表格切卡片 + modal 全屏）
- 修：API 调用 `apiFetch` → `API.get/post`（fnclearup 用的是 `API.base` 模式）
- 修：API.base 末尾斜杠导致双斜杠路径 404
- 修：诊断 BUILD 版本号 + 时间戳（防止 fnOS 僵尸旧 server 进程）
- 修：trash 不区分用户，root 包含所有用户的 trash（撤销 otherTrash 灰化）
- 测试 60/60 通过

### v0.7.9 (2026-06-02)

- 修复：空目录扫描期间浏览/stop 无响应（walk 同步阻塞事件循环）
- 修复：emptyDirJobs 注册时机（stop 提前到达能立即被处理）
- walk/collect 改为 async，每 200 目录 yield 一次
- 测试 60/60 通过

### v0.7.8 (2026-06-02)

- 新增启动警告弹窗（每次打开应用强制阅读 3 秒，红底警示 + 强制确认项 + manifest 开关推荐）
- 防止用户没看清就点删除

### v0.7.7 (2026-06-02)

- Manifest 写入位置改到 `<TRM_PKGVAR>/data/manifests/`（与 info.log 同父目录，fnOS 标准 data/ 子目录）
- `getSafeTrashDir` → `getManifestDir` 重命名（保留旧名 alias）
- 删除 UI 上的"写到 .@#local/"提示文案
- 测试 58/58 通过（+2 路径验证）

### v0.7.6 (2026-06-02)

- 新增「系统缓存清理」Tab（14 项可清理：APT/系统日志/NPM/PIP/UV/TypeScript/浏览器缓存/应用日志/旧内核等）
- 「✅ 推荐清理」一键勾选 10 项低风险 auto-rebuild 安全项
- 「📦 清理前备份 manifest」开关（默认 ON，写到 `<TRM_PKGVAR>/data/manifests/fnclearup_pre_clean_manifest_<时间>.json`，与 info.log 同父目录）
- 「📜 历史清单」弹窗，列出/下载历史 manifest（审计用）
- 新增 `getManifestDir()` 工具函数（按 `TRM_PKGVAR`/test/fallback 优先级返回 `<PKGVAR>/data/manifests`，与 `info.log` 同父目录）
- 新增 API：`GET /api/sysclean/manifests`、`GET /api/sysclean/manifest?path=...`
- 路径遍历防护：manifest 下载端点强制白名单
- 测试 58/58 通过（+6 个新用例：备份写入/不写/列表/下载/路径遍历拒/缺参拒 + 2 个路径验证）

### v0.7.5 (2026-06-02)

- 修复 5 个 modal「确认」按钮 onclick 调错函数（confirmXxx → doXxx）
- P0/P1 安全修复：4 个删除端点 isSafePath 白名单、handleBrowse /vol* 白名单、CSV 注入防护
- 删除用户强警告（双重 confirm）+ 3 个 docker 删除加 modal 确认
- empty_dir 重写：stop API、跳过 @app*、流式扫描、state file 修复 complete 状态保留
- 删除 app/data/ 隐私数据 + app/ui/utils.js 冗余代码
- CORS 改 CORS_ORIGINS 环境变量
- 测试 39/39 通过（新增路径遍历、CSV 注入、HTML onclick 契约测试）

### v0.7.2 (2026-05-30)

- 新增「去空」选项卡 — 扫描空目录，支持批量/单个删除（回收站/永久删除）
- 实时扫描目录显示 + 进度条
- 移动端自适应（空目录列表水平滚动 + 路径自动换行）
- 单遍遍历算法优化 + 符号链接安全（lstatSync）
- uncaughtException + 进程退出码检测，错误写到状态文件
- 跳过 `.@#local` 回收站目录
- 版本号统一为 0.7.2

### v0.7.1 (2026-05-29)

- 新增 tmp 选项卡（独立版本前已有，版本号对齐）

### v0.7.0 (2026-05-26)

- 新增文件去重模块（SHA-256 哈希比对，10 线程并行计算）
- 新增音乐去重模块（基于元数据的智能重复识别）
- 新增目录浏览选择按钮（内置文件选择器）
- 新增顶部版本号显示，启动时自动检查 FnDepot 是否有新版本
- 去重模块移除暂停/继续按钮，仅保留停止按钮
- 去重 Tab 标签精简：重复 → 去重，音乐重复 → 音乐，文件重复 → 文件
- 回收站路径纠正为 `/volX/UID/.@#local/trash/`
- 删除失败时显示具体错误信息
- 修复：组删除排除 disabled 原始文件
- 修复：暂停/继续功能完善（checkPause 先同步文件再检查）
- 修复：赞助图片放大 onclick 自动转换导致 this 丢失
- 修复：状态文件写入改为原子操作，避免并发读导致 JSON 解析失败
- 修复：移动端长内容截断

### v0.6.0 (2026-05-05)

- Docker 网络清理功能完善（tab 头部删除按钮恢复）
- 新增系统资源概览 `/api/system` 端点（磁盘分区、内存状态、Docker 基础信息）
- 应用/网盘选项卡合并表格：状态列+badge，参照 docker 表格样式
- 移动端 KPI 卡片保持 3 列自适应，缩小图标和文字
- 修复：每次打开强制重新加载 main.js（添加时间戳参数避免缓存）
- 修复：应用 tab 和网络 tab 状态文字格式优化

### v0.5.6 (2026-05-03)

- 移动端 KPI 响应式布局（768px 以上 3 列，480px 以下堆叠为 3 行）

### v0.5.5 (2026-05-03)

- 移动端 KPI 恢复 3 列布局

### v0.5.4 (2026-05-03)

- 修复 main.js 语法错误（JS 执行失败导致按钮无效）
- 修复展开按钮初始无箭头问题
- 修复数据无法加载问题

### v0.3.0 (2026-05-01)

- 前端全面优化：无障碍修复（aria-label、role="dialog"）、模态框标注、alert 阻塞修复
- 新增：网盘 Tab 新增"目录总数"KPI 卡片
- 修复：do_volumes 中 orphan_json 缺少 [] 包裹导致前端 "not iterable" 错误

### v0.2.0 (2026-04-30)

- 新增 Tab 切换：网盘挂载目录 / data/vol02 未挂载目录独立展示
- 新增 KPI 卡片：显示已挂载数量和未挂载数量

### v0.1.5 (2026-04-30)

- 前端审计修复: XSS防护(innerHTML转义)、label for属性、WCAG AA对比度优化

## 维护者

- 作者：[@再见一零一二](https://gitee.com/wyf1015)

---

> 如果这个项目对您有帮助，欢迎赞助支持 ❤️
