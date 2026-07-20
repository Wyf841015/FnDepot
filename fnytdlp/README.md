# fnytdlp - 飞牛 NAS 视频下载器

> 集成 [yt-dlp](https://github.com/yt-dlp/yt-dlp) (1872+ 站点) 的 fnOS 视频下载工具，零依赖、纯 Node.js。

![fnytdlp 视频下载器](ICON_256.PNG)

## 功能特性

- **1000+ 视频站**：YouTube / B 站 / 抖音 / 微博 / X / 西瓜 / 推特 / Twitch / Vimeo / 任意 yt-dlp 支持的站
- **yt-dlp 完整参数**：格式选择、字幕下载、缩略图、SponsorBlock 广告跳过、元数据嵌入、Cookie 认证、代理
- **自适应架构**：x86_64 + aarch64 双 binary 打包，启动时按 `process.arch` 自动选
  - 当前 zipimport 模式 binary 是平台无关 Python 脚本（同一份内容两架构都能跑），但保留双文件方便未来替换为 musl native binary
  - 兜底逻辑：env `YT_DLP_BIN` > 架构匹配 > 通用 `yt-dlp`
- **统一网关模式**：Unix socket + HTTP 双端口，fnOS 网关原生代理
- **现代 UI**：8 项 UI 重构（KPI Dashboard / Toolbar / Task Card / Settings sidebar / Cookie modal / 5 主题）
- **实时进度**：SSE 推送 + 2s 轮询双备份，实时显示速度/ETA/进度
- **任务详情**：点击任务卡弹窗显示 ID/标题/URL/文件名/大小/已下载/存储位置/状态/进度/格式/创建/完成/错误等 13 字段
- **下载路径浏览**：设置面板支持 📂 按钮 + 弹窗选择目录（不限制白名单，可任意浏览后选择）
- **Cookie 多网站管理**：单文件 → 数组结构 + 任务级 cookieName 透传，浏览器扩展导入
- **页脚版权**：动态版本号 + 自动年份，yt-dlp 加原项目链接
- **安全**：SSRF 防护（只 http/https）/ 路径白名单 / Cookie 加密存储 / fetch 30s 超时 / 错误脱敏
- **零运行时依赖**：纯 Node.js ESM 模块，fpk 解包即跑
- **磁盘配额** (v0.4.0)：下载目录容量上限，超额自动清理最旧已完成任务
- **下载历史统计** (v0.4.0)：4 个累计 KPI + 双 canvas 图表 (每日/按域名)，v0.4.1 移到独立 modal
- **文件名模板预览** (v0.4.0)：输入 `%(title)s` 实时显示展开效果
- **yt-dlp 高级参数** (v0.5.0)：视频裁剪 / aria2c / 转码 / 限速模板 / 主题跟随系统 / 导入配置 / 任务标签 / 重复检测 / 文件名预设
- **AI 视频总结** (v0.6.0)：4 提供商 / 4-Tab 展示 / 思维导图 / 缩略图代理 / 字幕提取 / 速度曲线

## 安装

### 在 FnOS 应用中心安装

1. 下载 `fnytdlp.fpk`（约 6.3 MB，v0.6.0）
2. 在飞牛NAS应用中心 → 手动安装
3. 首次进入"设置"，配置下载路径（建议 `/vol2/1000/fnytdlp/`）
4. 点"+"粘贴视频链接开始下载

### 直接运行

```bash
cd /path/to/fnytdlp
PORT=19634 node ui/server.js
```

然后访问 `http://localhost:19634` 即可。

## 10 大核心参数

| 参数 | 作用 | 默认 |
|------|------|------|
| `--format` | 格式选择 (bv*+ba/b) | bv*+ba/b |
| `--output` | 文件名模板 | `%(title)s [%(id)s].%(ext)s` |
| `--cookies` | Cookie 文件 (会员内容) | 关闭 |
| `--write-subs` | 下载字幕 | 关闭 |
| `--write-thumbnail` | 下载缩略图 | 开启 |
| `--sponsorblock-mark` | 标记广告片段 | 关闭 |
| `--embed-metadata` | 嵌入标题/上传者/日期 | 开启 |
| `--no-playlist` | 单视频 (不下载播放列表) | 关闭 |
| `--concurrent-fragments` | 分片并发数 | 4 |
| `--retries` | 重试次数 | 3 |

## Cookie 认证

YouTube 会员 / B 站大会员等需要登录才能看的视频：

1. 浏览器装 "Get cookies.txt LOCALLY" 扩展
2. 登录目标站 → 导出 cookies.txt
3. 应用里点 🍪 Cookie → 粘贴 → 保存
4. 下次下载自动加载

### 多网站 Cookie

支持同时管理多个站点的 Cookie（YouTube 会员、B 站大会员、抖音登录等），任务级 `cookieName` 字段透传指定。

## SponsorBlock

自动跳过视频里的广告/赞助/自我介绍片段：

- 设置 → 高级 → `sponsor,selfpromo,intro,outro` (用逗号分隔)
- 标记的片段会写入视频元数据，播放器识别后自动跳过

## 实时监控

- KPI Dashboard：4 张卡片显示总任务/下载中/已完成/失败 + 实时速度/ETA/已用时间/平均速度
- SSE 实时推送：进度变化无延迟，配合 2s 轮询兜底
- 速度小图：6 个采样点滑动窗口绘制（Sparkline）

## 频道订阅

自动监控 YouTube 频道 / B 站 UP 主 / 播放列表等，定时拉取新视频并加入下载队列。

### 表格化订阅管理（v0.3.1）

订阅列表从横向 chip 改为**表格形式**，每行清晰展示：

| | 名称 | URL | Cookie | 间隔 | 格式 | 最新ID | 操作 |
|---|------|-----|--------|------|------|--------|------|
| 开关 | 频道名 | https://youtube.com/@xxx | cookie-name | 60 分钟 | bv*+ba/b | 上次下载的 id | 🔍 立即检查 / 🗑 删除 |

- **iOS 风格开关**：绿色 = 启用，灰色 = 暂停，暂停时整行 0.55 opacity 灰显
- **每行独立检查**：点击 🔍 跳过 5 分钟节流立即检查该订阅（其他订阅不受影响）
- **增量下载**：基于 `lastId` + 时间窗口双保险，只下载新内容不重下历史
- **数据校验**：item.url 为空时 LOG 跳过，避免静默重新下载整个 playlist

### 增量下载策略（v0.3.1）

```
新内容 = 时间戳 > 上次检查时间 的项
兜底：lastId 失效（视频被删/下架）→ 自动切到时间窗口
```

| 策略 | 触发 | 行为 |
|------|------|------|
| **lastId 命中** | 频道里能找到上次下载的 id | 取 lastId 之前的全部 = 新内容 |
| **时间窗口兜底** | lastId 找不到（视频被删） | cutoff = lastCheckAt - 1h，保留 > cutoff 的项 |
| **id 去重** | yt-dlp 返回重复项 | 按 videoId 去重，避免重复加任务 |
| **URL 校验** | dump-json 拿不到完整 url | 跳过该条并 LOG，避免 fallback 到 playlist URL 引发重下 |

### 添加订阅

1. 点 🔔 订阅按钮 → 在弹窗"添加新订阅"折叠区填：
   - 频道名称（任意）
   - 频道 URL（YouTube `@xxx`、B 站 UP 主主页、播放列表）
   - Cookie（可选，已保存的 cookie 名称）
   - 检查间隔（30 分钟/1 小时/2 小时/6 小时/24 小时）
   - 格式（可选，留空用全局默认）
2. 点 💾 保存订阅，自动加入检查队列
3. 每 5 分钟（fnOS 内置 cron）自动检查所有启用的订阅

### 常见场景

- **大频道 1000+ 视频**：默认 `--playlist-end 500` 限制单次取 500 项 + 时间窗口兜底，避免重下历史
- **视频被上传者删除**：lastId 失效自动切时间窗口，不会卡死
- **重复添加同名订阅**：后端去重（同名/同 URL 覆盖），保留运行时字段（enabled/lastId/addedAt）

## 项目结构

```
fnytdlp/
├── ui/                          # 前端 + Node.js server
│   ├── server.js                # 后端 HTTP+UNIX socket (API 路由 + 任务管理)
│   ├── main.js                  # 前端逻辑 (任务管理 / 主题 / 设置 / 搜索 / 赞助 / 详情)
│   ├── index.html               # 入口 (响应式 + 5 主题 + 移动端适配)
│   ├── styles/                  # HSL 设计系统 (tokens/components/layout)
│   │   ├── tokens.css          # CSS 变量 (HSL 主色/灰阶/间距/圆角/阴影)
│   │   ├── components.css      # 组件样式 (button/card/modal/sponsor 等)
│   │   └── layout.css          # 布局 (header/main/aside/footer)
│   ├── util/
│   │   ├── sparkline.js        # KPI 趋势小图 (ESM 模块)
│   │   └── url-resolver.js     # 动态直播源解析 (PHP 路径嗅探 + SSRF 守卫)
│   └── bin/
│       ├── yt-dlp-x86_64        # x86_64 平台 (fnOS 服务器主流)
│       ├── yt-dlp-aarch64       # aarch64 平台 (ARM NAS)
│       └── LICENSE.txt          # yt-dlp Unlicense 全文
├── tests/                       # TDD 测试套件 (104 个测试)
│   ├── test_comprehensive.js           # 基础功能测试 (parseSpeed/parseDuration 等)
│   ├── test_comprehensive_server.js    # 服务端+前端纯函数测试 (59 个用例)
│   └── test_progress_aggregator.js     # 多流进度聚合测试 (12 个场景)
├── cmd/
│   ├── bin/yt-dlp-x86_64        # 跨平台 yt-dlp binary
│   ├── bin/yt-dlp-aarch64
│   └── main                     # fnpack 启动脚本
├── config/
│   ├── privilege                # run-as 用户配置
│   └── resource                 # 资源配额
├── wizard/uninstall             # 卸载向导
├── manifest                     # fnpack 配置
└── fnytdlp.fpk                  # 打包产物 (6.3 MB)
```

## API 端点

| 端点 | 方法 | 描述 |
|------|------|------|
| `/api/health` | GET | 健康检查（含 arch/ytDlpBin/version） |
| `/api/config` | GET | 获取配置 |
| `/api/config` | POST | 保存配置（下载路径/Cookie 目录/yt-dlp 参数） |
| `/api/tasks` | GET | 获取任务列表 |
| `/api/tasks` | POST | 添加下载任务（支持 options 透传） |
| `/api/tasks/:id` | GET | 获取单个任务 |
| `/api/tasks/:id` | DELETE | 删除任务（支持 `?deleteFiles=1`） |
| `/api/parse` | GET | 解析视频 URL 提取元数据 |
| `/api/browse` | GET | 浏览目录（白名单子目录列表） |
| `/api/cookies` | GET/POST/DELETE | Cookie 多文件管理 |
| `/api/subscriptions` | GET/POST | 订阅列表/保存（POST 含 toggle enabled） |
| `/api/subscriptions/:name` | DELETE | 删除订阅 |
| `/api/subscriptions/check` | POST | 手动触发订阅检查（body 可选 `{name}` 单订阅） |
| `/api/events` | GET | SSE 实时进度推送 |
| `/api/progress` | GET | 任务进度（保留兼容） |

## 数据目录

应用运行时数据存放在 `${TRM_PKGVAR}/`：

- `config.json` - 用户设置
- `tasks.json` - 任务列表
- `cookies/` - 多 Cookie 文件目录 (mode 0600)
- `info.log` - fnOS 框架日志
- `data/downloads/` - 默认下载目录
- `data/manifests/` - 应用状态/审计数据

## 系统依赖

- Node.js 20+ (fnOS 框架自带)
- Python 3.7+ (yt-dlp zipimport 模式需要)
- ffmpeg (`/usr/bin/ffmpeg`，fnOS 系统内置 7.1.3-mediassr 带 VAAPI/CUDA 加速)

## 开发

```bash
# 启动测试服务器
TRM_PKGVAR=/tmp/fnytdlp-test PORT=19634 node ui/server.js

# 跑全部测试
node --test tests/*.js

# 打包
fnpack build .
```

## 架构亮点

### 跨架构 Binary 选择

`process.arch` (Go 风格: x64/arm64/ia32) + 4 优先级 (env `YT_DLP_BIN` > arch 匹配 > 通用 `yt-dlp`)。zipimport 模式 6.4MB 总，比 musl native 56MB 省 88% 体积，GitHub 下载快 10x。

### 动态直播源解析

PHP 动态直播源（如 `http://example.com/live.php?id=xxx`）自动检测 + 嗅探（URL 扩展名 / Content-Type / body 头 `#EXTM3U`）+ SSRF 5 层守卫 + redirect 二次验证。url-resolver.js 145 行 ESM 零依赖。

### 周期轮询 3 层架构

`initXxxDashboard` (幂等) / `loadTasks` (全量，结构变化) / `pollTasks` (增量，改 textContent 不 innerHTML)。比 `setInterval(loadTasks)` 强：无 focus 丢失 / 无 scroll 位置重置 / GPU 加速。

## 版本历史

### v0.6.0 (2026-07-20)

**AI 视频总结 + 缩略图代理 + 字幕提取 + 速度曲线 (借鉴 uvd 项目)**

- **🧠 AI 视频总结** (零依赖, Node 22 fetch 调 OpenAI 兼容 API)
  - 支持 4 家提供商: OpenAI / 智谱 GLM / DeepSeek / 自定义
  - yt-dlp 自动提取字幕 (--skip-download --write-subs), fallback 到 description
  - 4-Tab 展示: 智能总结 / 章节大纲 / 核心要点 / 思维导图
  - 思维导图零依赖渲染 (Markdown 嵌套列表 → 树状 HTML), 无需 markmap CDN
  - 复制 Markdown 全文到剪贴板
  - 配置: 设置 → AI tab (aiEnabled/aiProvider/aiBaseUrl/aiApiKey/aiModel); 环境变量 AI_API_KEY 覆盖
- **缩略图代理** (GET /api/proxy-thumbnail?url=...) — 解决防盗链/跨域, 抖音 CDN 自动加 Referer
- **字幕提取** (GET /api/tasks/:id/subtitle) — VTT/SRT → 纯文本, 按 zh-Hans/zh/en 优先级匹配, 任务详情 📝 按钮
- **单任务速度曲线** — task._speedHistory 200 采样点, 任务详情 canvas 折线, SSE 实时刷新
- **URL 预处理** — 抖音 modal_id/note URL 自动转标准 video 格式 (normalizeUrl)
- 新增 5 API 端点: /api/ai/summarize / /api/ai/progress/:id / /api/ai/result/:id / /api/proxy-thumbnail / /api/tasks/:id/subtitle
- 测试 221 → 273 (+52), 6 个新 describe 块覆盖全部新功能

### v0.5.0 (2026-07-20)

**8 项新功能 (yt-dlp 高级参数 + 体验增强)**

- **视频裁剪** (`--download-sections` + `--force-keyframes-at-cuts`) — 截取中间片段, B 站长视频常用
- **aria2c 外部下载器** (auto/always/never) — 大文件 16 连接提速, 自动检测 fnOS 系统 aria2c
- **视频转码** (`--recode-video mp4`) — 下载完成后自动混流到指定容器
- **限速模板** (按时段自动切换, 支持跨天) — 例: 22:00-07:00 限速 10M, 白天不限
- **任务按格式筛选 tab** (🎬 视频 / 🎵 音频) — 按 ext 自动归类
- **任务标签** (逗号分隔, 最多 10 个) — 复杂场景筛选
- **重复下载检测** (`--download-archive` 启用时) — POST /api/tasks 检查 archive, 已下载返回 `{duplicate: true}`
- **导入 yt-dlp.conf** (30 参数白名单 + 注释过滤) — 老用户配置一键迁移
- **主题跟随系统** (`prefers-color-scheme`) + **ETA 详细显示** (`01h23m45s`)
- **yt-dlp GitHub 更新检查** (6h 缓存, 启动后台触发) + **一键复制 yt-dlp 命令**
- **文件名模板预设** (默认/按作者分目录/仅标题/播放列表编号)
- 新增 2 API 端点: `GET /api/yt-dlp/check-update` + `POST /api/config/import-yt-dlp-conf`
- 测试 154 → 221 (+67), 端到端验证: `/api/health` aria2cExists ✓ + archive duplicate ✓ + yt-dlp.conf import 4 参数 ✓

### v0.4.1 (2026-07-20)

**UI 调整: 统计 tab 从设置菜单移到独立 modal**

- 把 v0.4.0 的「📊 统计」从 settings 内嵌 tab 拆成独立 modal
- toolbar 末尾新增 📊 统计按钮 (跟 ⚙ 设置 / 🍪 Cookie / 🔔 订阅同一级别)
- settings tabs 减少: ⚙ 基本 / 🔧 高级 / 🔍 过滤 / 💾 存储 (4 项)
- 独立 modal 设计: modal-header + 4 个累计 KPI + 双 canvas 图表, 关闭按钮
- 打开统计 modal 立即渲染一次 renderStatsPanel

### v0.4.0 (2026-07-20)

**磁盘配额 + 历史统计 + 文件名模板预览 + 搜索增强**

- **磁盘配额** (新增 💾 存储 tab)：设置下载目录容量上限 (例 `50G`)，支持立即检查用量 + 超额时自动清理最旧已完成任务
- **下载历史统计** (新增 📊 统计 tab)：4 个累计 KPI (总下载 / 完成任务 / 本月下载 / 总耗时) + 双 canvas 图表 (近 30 天每日下载量 + 按域名分布)
- **文件名模板实时预览**：输入 `%(title)s [%(id)s].%(ext)s` 即时显示"展开效果"（基于已完成任务的真实样本）
- **搜索字段扩展**：搜索框同时匹配 URL + 文件名 + 标题（之前 filename 优先会漏掉 title）
- 测试 104 → 152 (新增 48 个测试覆盖 parseSizeString / API 路由 / 函数存在性 / HTML 结构 / CSS 规则 / 实时预览逻辑)

### v0.2.4 (2026-07-07)

**性能优化 & 代码清理**

- SSE 增量更新: task-created/task-updated 改为增量合并, 不再全量 loadTasks() (下载中场景 ~80% HTTP 请求减少)
- 后备轮询 5s→30s (SSE 为主, 轮询备)
- serveStatic 流式发送: >512KB 文件用 createReadStream (避免 yt-dlp binary 全量占内存)
- 删除重复的 yt-dlp 版本检查 (启动时只检查一次)
- 内联 CSS (48 行) 从 index.html 迁移到 components.css

### v0.2.3 (2026-07-07)

**测试覆盖增强**

- 新增 59 个纯函数测试 (`test_comprehensive_server.js`)，3 个测试文件共 104 个用例全部通过
- **服务端安全函数**：`sanitizeFilename`(路径穿越防御/128截断/null兜底)、`isValidUrl`(SSRF防护/file/ftp/data/js拦截)、`isSystemPath`(11个系统路径阻断)、`isSafeDownloadPath`(白名单校验)、`safeName`(Cookie文件名安全化)、`detectArch`(x64/arm64/ia32)、`validateCookieContent`(Netscape格式校验/100KB限制)
- **API 逻辑**：yt-dlp `-F` 格式解析(avc1/VP9/HEVC/Opus 6种)、batch URL去重、任务去重(downloading去重/completed放行)、Cookie域名匹配(含短域名误匹配防护)、parseBody 2MB限制
- **前端纯函数**：`formatBytes`(KB/MB/GB)、`formatSpeed`、`formatDuration`(MM:SS/HH:MM:SS)、`esc`(HTML 5字符转义)
- 项目结构 `tests/` 目录更新为3个测试文件

### v0.2.2 (2026-06-11)

**安全加固**

- 代码审计修复（P0-P2 共 9 项）：空 catch 日志化 / 全局异常处理器 / API 原子写入 / 系统路径黑名单 / spawn 24h 超时 / browse 路径校验 / 批量操作异常捕获 / confirmModal 按钮禁用 / fetch 15s 超时
- SSRF 防护 + 路径白名单 + Cookie 加密存储 + 错误脱敏

### v0.2.5 (2026-06-11)

**Bug 修复**

- 多流下载友好界面：视频/音频两段子进度条（video cap 50% 防"两次 100%"），合并阶段显示 99%
- 临时文件名暂存：`xxx.f100026.mp4` 分片不覆盖 `task.filename`，合并后最终 `.mp4` 才展示
- 任务详情存储位置包含子目录：路径改为 `downloadPath/video_xxx/xxx.mp4`
- 格式显示改为人类可读：`100026+30280` → `1080p HEVC · mp4 + 128k mp4a · m4a`
- 部署目录权限修复：新加文件使用 `install -m 644 -o package -g package` 替代 cp

**新功能**

- `describeFormatIds()` 反查 formats 数组，formatId → 分辨率/编码/码率
- `showTaskDetail` 优先显示 `formatDescription`

### v0.2.0 (2026-06-10)

**Bug 修复**

- 文件名显示封面图：`[download] Destination:` 解析时过滤 `.webp/.jpg/.png/.gif` 封面扩展名；已完成旧任务兜底扫描也检查 filename 是否为封面图
- 工具栏清理按钮无确认框：`clearCompleted` 增加 `showConfirm` 弹窗
- 赞助大图点击不放大：inline onclick 重绑定不支持 `this` 关键字，箭头函数改 `function` + `thisArg`

**安全加固**

- 清理已完成任务前增加确认弹窗，防止误操作

### v0.1.0 (2026-06-09)

**核心功能**

- 集成 yt-dlp 1872+ 站点支持
- 自适应 x86_64 + aarch64 双 binary
- 4 张 KPI 卡片 + Sparkline 实时趋势
- SSE 实时进度推送 + 2s 轮询兜底
- 任务详情弹窗（13 字段含 ID/URL/文件名/大小/已下载/存储位置等）
- 下载路径浏览对话框（📂 按钮 + 弹窗选择任意目录）
- 多网站 Cookie 任务级透传
- SponsorBlock 广告跳过 / 字幕 / 缩略图 / 元数据嵌入
- 5 主题 + 明暗模式 + 响应式布局（桌面/移动端）
- SSRF 防护 / 路径白名单 / Cookie 加密存储 / 30s fetch 超时
- 20+ TDD 测试套件，集成 / 单元 / 端到端覆盖

**Bug 修复**

- sparkline.js ESM 模块加载：script 标签加 `type="module"` + main.js 用 `import`
- 实时速度 KPI 换行：字号 1.1→0.95rem + `white-space: nowrap` + ellipsis
- `r.pipeThrough is not a function` TypeError 500：ArrayBuffer 没 pipeThrough API 误用修复
- 空状态两个按钮等宽居中：`flex: 1` 替代 min-width
- 任务列表无闪烁刷新：去 spinner + SSE rAF 节流 + 5s 轮询 + CSS contain
- 版本号统一从 manifest → 改为 VERSION 常量（启动速度 + 0 失败）

**UI 优化**

- 8 项 UI 重构（KPI Dashboard / Toolbar / Task Card / Settings sidebar / Cookie modal / Batch bar / Empty state / Sparkline）
- 移动端 5 项适配（KPI 2 列 / Header 隐藏时钟 / 工具栏横滑 / Batch bar 紧凑 / 弹窗全屏）
- 工具栏 ghost 按钮背景色统一（var(--bg) 浅底 + emoji 字号规范）
- 页脚版权信息：动态版本号 + 自动年份 + yt-dlp / FnDepot 链接

**安全加固**

- CORS 白名单同源回退
- file:// URL 400 而非 500
- IPv4 映射 IPv6 内网地址绕过修复
- url-resolver 流式 body 限 1MB
- 错误信息路径泄露过滤

## 测试

```bash
# 运行全部测试 (104 个)
node --test tests/*.js

# 运行特定测试文件
node --test tests/test_comprehensive.js        # 基础功能 (33 个)
node --test tests/test_comprehensive_server.js # 服务端/前端纯函数 (59 个)
node --test tests/test_progress_aggregator.js  # 多流进度聚合 (12 个)
```
## 维护者

- 作者：[@再见一零一二](https://gitee.com/wyf1015)
- GitHub：[@Wyf841015](https://github.com/Wyf841015)
- Gitee：[@wyf1015](https://gitee.com/wyf1015)
- 第三方依赖：[yt-dlp](https://github.com/yt-dlp/yt-dlp) (Unlicense)

---

> 如果这个项目对您有帮助，欢迎赞助支持 ❤️
> 本应用打包了 [yt-dlp](https://github.com/yt-dlp/yt-dlp)（Unlicense 公共领域），保留完整 LICENSE 文本。

## 更新日志

### v0.3.1 (2026-07-10)

**订阅功能增强 & 增量逻辑修复**

订阅 UI 重构：
- 订阅列表从横向 chip 改为**表格形式**（8 列：开关/名称/URL/Cookie/间隔/格式/最新ID/操作）
- **iOS 风格开关**（绿色=启用/灰色=暂停），暂停时整行 0.55 opacity 灰显
- 每行独立 **🔍 立即检查**按钮（绕过 5 分钟节流）
- 删除按钮 hover 红色高亮

订阅增量逻辑修复（5 个 bug）：
- **lastId 卡死**：lastId 对应视频被删/下架时找不到了，原逻辑把所有项当新内容，lastId 永远卡死 → 修复为 fallback 到时间窗口
- **多页列表重下**：大频道 1000+ 视频时 playlist 只显示前 100，原逻辑会把 lastId 之前所有项当"新" → 时间窗口兜底只保留 lastCheckAt 之后的项
- **顺序假设**：`--playlist-reverse` 在新版 yt-dlp 被忽略 → 改用 timestamp 字段（timestamp/release_timestamp/upload_date）主动排序
- **重复添加**：yt-dlp flat-playlist 偶尔返回重复 id → 按 id 去重
- **URL 兜底重下**：item.url 为空时 fallback 到订阅 URL → 改为 LOG 跳过，避免静默重下整个 playlist

订阅后端 bug 修复：
- `POST /api/subscriptions` 之前会强制把已有订阅 `enabled` 重置为 `true`（toggleSubscription 失效） → 修复为新建用默认，已有保留原 `enabled/lastId/addedAt`
- 新增 `parseBodySafe` 容错版（空 body 返 `{}` 而非 reject）

新增功能：
- `POST /api/subscriptions/check` body 支持 `{name}` 单订阅检查
- 单订阅检查强制绕过 `interval` 节流，立即生效
- `sub._lastCheckAt` 记录时间窗口 fallback 用时间戳
- `--playlist-end 500` 限制单次最多取 500 项（大频道保护）
- getLatestIds timeout 30s → 60s

单元测试：新增 9 个增量逻辑测试（首次检查 / lastId 命中 / lastId 失效 fallback / 时间窗口边界 / 去重 / 顺序）

### v0.3.0 (2026-07-09)
- UI 品牌色重绘：暖橙→亮蓝+青绿+暖金科技冷色系
- 搜索栏与筛选标签合并为一行（inline search）
- 任务卡片重排：状态徽章移入标题行，meta 行紧凑分隔
- 添加任务弹窗改进：URL 卡片化+实时验证+格式 pills
- 空状态增强：光晕动效+快捷键提示
- prefers-reduced-motion 动效尊重（系统级无障碍）
- 按钮加载态：submitAddTask 防双击
- 移动端 touch target ≥ 44px
- 语义化标题层级（h1/h2）
- 搜索无结果/错误恢复提示
- 主题覆盖全面修复（12 处老配色/缺失类）

### v0.2.4 (2026-07-07)
- 性能优化: SSE 增量更新(HTTP请求~80%减少) / 后备轮询 5s→30s
- serveStatic 流式发送 >512KB 文件用 createReadStream
- 删除重复 yt-dlp 版本检查 / 内联 CSS 迁移到组件文件
