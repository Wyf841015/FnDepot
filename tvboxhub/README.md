# TvBox For Emby

> 把 TVBox 的点播源接进 Emby：跨站搜索、一键推送 strm、按站点建媒体库，内置 JRE 开箱即用。

![TvBox For Emby](ICON_256.PNG)

## v0.3.0 更新

**外网播放：strm 可以写源站直链**

- 🐛 问题：strm 里是内网地址（`http://192.168.x.x:8096/TvBox/Play?…`），外网播放器根本连不上；走 fnOS 网关又要登录态，第三方播放器拿不到 cookie
- ✅ 新增「strm 播放地址方式」：
  - `代理` —— 原行为，内网最稳
  - `直链` —— 写源站直链，客户端直连源站，**外网可播且不占 NAS 带宽**
  - `直链优先` —— 解析不出真实地址的集退回代理
- ✅ 新增「播放地址基址」：填域名可做内外分流（家里 DNS 指内网、外网指公网），一份 strm 内外通吃
- ✅ 新增「直链重解析间隔」（默认 12 小时）：源站 `auth_key` 会过期，到点自动重新解析刷新 strm
- ⚡ 直链解析并发化（`并发数 × 2`，上限 8）
- 实测：裸连直链（无 UA、无 Referer）m3u8 200 / 69KB，分片 200 / 62KB

## v0.2.1 更新

**直链缓存修正确 + 真实同步实测**

- 🐛 修复：同步开始时清空直链缓存 —— 上一轮预热的地址全废；若第二轮没有需要校验的分集，缓存会一直是空的
- 现在只更新 TTL、不清空（条目自带过期时间，密钥轮换不影响已缓存直链）
- 实测（单集 m3u8，286KB）：首播 **11.0s** → 命中缓存 **0.36s**（约 30 倍）
- 实测同步：192 条目 / 43 分集 / 0 错误；第二轮 12s，192 条全部「内容未变」
- 目录命名 74/76 带 `(年份)`，其余 2 个源站未提供年份（模板自动去掉空括号）

**v0.2.0 起新增的 12 项能力**（详见下文"功能特性"）

- 直链缓存与预热、播放签名（公网暴露防白嫖）、Webhook 三种事件、Emby 删除同步
- 命名模板 / 重命名正则 / 跳过关键词 / 字幕落地
- TMDB 规范化（只改 nfo 不改目录名）、时间戳增量、多计划调度、站点失败自动熔断

## v0.1.1 更新

**首个对外版本：点播源 → Emby 媒体库**

- 填 TVBox 配置地址（单仓 JSON / 多仓 urls / base64 / 本机绝对路径）即可采集站点
- 站点页批量探测，**只把「已确认可播」的站点加入同步**（有列表 ≠ 能播，实测不少源的播放地址返回的是解析接口 JSON）
- 跨站并发搜索 → 详情 → 试播 →「一键推送」，单条也能直接进库
- 同步按站点目录落盘，生成 `.strm` + `.nfo` + 海报，Emby 扫描后就是普通媒体库
- 播放由插件代理：自动补 `Referer` / `User-Agent`、重写 HLS 分片，防盗链的源也能播

**目录结构（本版定型）**

```
<输出目录>/<站点名>/movies/<片名 (年份)>/<片名 (年份)>.strm
<输出目录>/<站点名>/tv/<剧名 (年份)>/Season 01/<剧名 (年份)> - S01E01 - <集名>.strm
```

站点层放在 `movies` / `tv` 的**外侧** —— Emby 靠「含 Season 目录的那层」认剧名，站点层放里面会让它拿站点名去刮削。
代价是同一部剧在多个站点下各存一份（来源清晰、互不覆盖）。加媒体库时按站点建：
电影 → `<输出目录>/<站点>/movies`，电视剧 → `<输出目录>/<站点>/tv`。

**🗑 删除历史数据（同步页新增）**

- 「先看看要删什么」只统计不删，弹窗显示目录结构、文件数、总大小、按扩展名分布
- 执行需输入 `DELETE`（小写不认），已列入破坏性接口，强制令牌 / 同源校验
- 边界写死：只进 `<站点>/movies`、`<站点>/tv` 两个子树，只删
  `.strm .nfo .srt .ass .vtt .jpg .jpeg .png .webp .tbn`，**不跟随也不删符号链接**，站点根目录保留

**🐛 修复**

- **搜索页推送把片名当站点目录**：`PushOneAsync` 写盘时第一个参数（站点名）传成了条目名称，
  推「集结号」会落到 `…/tvbox/集结号/movies/集结号 (2007)/`。现在由 hub 按 key 带 `siteName` 下去，
  取不到才退回 site key，**绝不再退到片名**；与全量同步的 `site.DisplayName` 完全一致
- **部署脚本指向不存在的 Emby 应用**：一直往旧应用目录装 DLL，而实际运行的是另一个 Emby，
  插件从未真正更新到运行中的 Emby 上（目录不存在 → 静默跳过）。部署脚本改为遍历 `/vol3/@appdata/Emby*`
  自动找带 `plugins/` 的应用，并按现有 DLL 属主 `install`
- **`autoDetectOutputRoot()` 正则多转义**：写成 `\\/` 导致 `/` 被当成正则结束符，整个函数此前跑不起来

**测试**：沙箱夹具全流程通过（预览 → 无令牌拒绝 → 缺 confirm 拒绝 → 删除 4 项 → 非目标文件全部保留 → 再预览 0）；
真实库只读预览 `layout=flat, files=2299`，推送实测落到 `金牌｜蓝光秒播/movies|tv/…`

## 功能特性

- **多格式采集源** — TVBox 单仓 JSON / 多仓 urls / base64 / 本机绝对路径
- **站点管理** — 站点列表 + 批量探测，「可播 / 未验证 / 不可播」分列标注
- **只要可播站点** — 批量加入同步严格只认 `playable === true`，无兜底；一个可播都没有时拒绝提交并提示先探测
- **跨站并发搜索** — 关键词一次打全部同步站点，结果按站点分组
- **一键推送** — 搜索/详情页把单条内容立刻写进库，与批量同步产物同名同结构
- **增量同步** — 按小时窗口跳过已同步条目；多线路源自动兜底换线
- **逐集校验** — 一个源常见「前几集能播、后面全是图片广告」，逐集验证后只写能播的那几集
- **落盘三件套** — `.strm` + `.nfo` + 海报（webp/jpg，带体积上限）
- **PlayUrl 解析** — 私有分集 id → 真播放地址（TVBox 客户端播放前做的同一件事）
- **孤儿清理** — 「我们写过什么」记在 `.tvbox-manifest.json`，只删清单里出现过、这次又不需要的文件
- **单次删除上限** — 一次要删超过 `MaxDeletePerRun`（默认 300）直接停手报错，防止命名规则一改把整库删空
- **删除历史数据** — 先统计、再输入 `DELETE` 确认，只删 9 类扩展名，站点根目录保留
- **三种流模式** — `proxy` 全中转 / `redirect` 直接 302 / `auto` 按域名白名单决定
- **HLS 分片重写** — 播放时把分片地址改回代理，防盗链源的 `.ts` 也能拉
- **Emby 插件一键装** — Web 端安装 / 卸载 / 自更新，装完自动重启 Emby 并显示重启进度弹窗（成功 1.6 秒自动关、失败保留原因）
- **重启弹窗可续接** — 中途刷新页面会自动接上并续算秒数
- **配置三入口对齐** — Web UI / 插件配置页 / API 三处 32 项字段完全一致，都落到同一份 `TvBox.xml`
- **配置体检** — 概览页 10 项自动检测，单独把 `redirect` 模式标红提醒（最容易配错又最难自查的一项）
- **使用向导** — 首次进入 7 屏向导 +「使用说明」里 6 项自动检测的进度清单
- **操作记录** — 安装、同步、清理逐条留痕
- **明暗主题** — 深色/浅色，服务端持久化（fnOS WebView 常禁用 localStorage）
- **响应式布局** — 侧边栏 8 页，移动端默认收起成抽屉；桌面 / 平板 / 手机三档宽度
- **内置运行时** — 包内自带裁剪版 JRE（x64 / arm64 各约 46 MB），不依赖系统 Java

## 快速上手

1. **配置 → 填点播源**：TVBox 配置地址（单仓 JSON / 多仓 / base64 / 本机路径）
2. **站点 → 批量探测当前**，勾选几个 ✅ 的源加入同步（建议先 3~5 个试水）
3. **配置 → 输出目录**：默认在 Emby 数据目录下的 `tvbox/`，也可指到自己的媒体盘
4. **Emby 插件 → 安装插件到 Emby**，完成后自动重启 Emby（弹窗显示进度，恢复后自动关闭）
5. **同步 → 开始同步**；跑完在 Emby 里把 `<输出目录>/<站点>/movies`、`…/tv` 加成媒体库并扫描

> 站点多的话可以把 `<输出目录>/<站点>` 加成「混合内容」库，一个站点一个库。

## 技术架构

```
TvBox For Emby

├── fpk-src/                     # fnOS 应用包
│   ├── cmd/                     # 生命周期脚本
│   │   ├── main                 # start / stop / status（status 退出码是 fnOS 判活的唯一依据）
│   │   ├── run-hub.sh           # 启动 hub（选 x64 / arm64 JRE）
│   │   ├── install_callback     # 补 JRE 执行权限 + 安装统计
│   │   ├── upgrade_callback     # 升级统计
│   │   ├── uninstall_callback   # 卸载回调
│   │   └── lib/feishu_push.sh   # 统计推送公共库（本机名单跳过）
│   ├── ui/
│   │   ├── index.html           # 单文件前端（侧边栏 8 页）
│   │   ├── hub.jar              # spider-hub 产物
│   │   ├── jre-x64 / jre-arm64  # 裁剪版 JRE
│   │   └── plugin/TvBox.dll     # Emby 插件产物
│   ├── config/                  # 权限配置（run-as: root）
│   └── manifest
├── spider-hub/                  # Java 中转服务
│   └── src/hub/
│       ├── Hub.java             # HTTP 44 路由 + 站点 / 搜索 / 同步 / purge
│       ├── SpiderPool.java      # catvod spider 运行时与站点池
│       └── …                    # MacCMS 适配层 / 播放解析
├── TvBox.Plugin/                # Emby 插件（C# / .NET 8，零 NuGet）
│   ├── Api/TvBoxApi.cs          # /TvBox/* 14 个路由
│   ├── Core/StrmWriter.cs       # 站点目录 + strm/nfo/海报写入 + 孤儿清理
│   ├── Core/SyncService.cs      # 同步主流程 / 单条推送
│   ├── Tasks/SyncTask.cs        # 定时同步 + 媒体库变更通知
│   └── Configuration/Web/       # 插件配置页（32 控件 / 5 分区）
├── spider-host/                 # catvod spider 可移植性实验舱（非运行时依赖）
├── deploy.sh                    # 部署：hub.jar + index.html + 插件 DLL，自动找活着的 Emby
└── build.sh                     # 编译插件（首次自动拉取 Emby 参考程序集）
```

- **后端**：Java（spider-hub，纯 JDK 无第三方依赖）；C# / .NET 8（Emby 插件，零 NuGet 依赖）
- **前端**：原生 HTML/CSS/JS，无框架依赖；播放用内联 hls.js
- **通信**：fnOS 网关 → UNIX Socket（`/vol3/@appcenter/tvboxhub/tvboxhub.sock`）；hub 的 TCP 只绑 `127.0.0.1`
- **数据存储**：`TvBox.xml`（配置单一真相源）+ `.tvbox-manifest.json`（写过什么）+ `.tvbox-state.json`（增量状态）
- **fnOS Gateway**：自动挂载 `/app/tvboxhub` 前缀
- **安全**：写接口需令牌或同源证明；安装 / 卸载 / 保存 / 清缓存 / 删文件列入破坏性白名单强制校验
- **兼容**：Emby Server 4.9 / 4.10（编译参考程序集同时带两套）

## API 端点

> 读接口用 GET；写接口（`/api/save`、`/api/reload`、`/api/push`、`/api/emby/*`、`/api/sync/purge`、
> `/api/cache/clear` 等）用 POST，且需带令牌或同源证明，否则直接拒绝。

| 端点 | 描述 |
|------|------|
| `/api/status` | 应用与 Emby 状态（含 `pending` 缺失项） |
| `/api/config` / `/api/save` / `/api/reload` | 读配置 / 写配置（33 项白名单）/ 重载 |
| `/api/prefs` | 界面偏好（主题、向导已读，服务端持久化） |
| `/api/sites` | 站点列表（含可播 / 未验证 / 不可播） |
| `/api/sites/ok` · `/api/sites/fail` | 批量标记站点探测结果 |
| `/api/sites/sync` · `/api/sync-sites` | 站点加入 / 移出同步 |
| `/api/probe` | 单站点探测（homeContent / 分类 / 列表） |
| `/api/home` · `/api/category` · `/api/detail` | 首页 / 分类 / 详情 |
| `/api/search` · `/api/search-all` | 单站搜索 / 跨站并发搜索 |
| `/api/play-resolve` | 私有分集 id → 真播放地址 |
| `/api/stream` · `/api/stream/seg` · `/api/go` | 播放代理 / 分片代理 / 跳转 |
| `/api/push` | 一键推送单条进 Emby 库（带 `siteName` 决定站点目录） |
| `/api/netdisk` | 网盘类源（默认排除） |
| `/api/history` · `/api/history/add` · `/api/history/clear` | 播放历史 |
| `/api/emby/detect` | Emby 5 路检测 |
| `/api/emby/status` | 含 `restart` 状态（active / done / ok / seconds） |
| `/api/emby/docker` | Docker 方式安装 / 管理 Emby |
| `/api/emby/install` · `/api/emby/uninstall` | 安装 / 卸载插件（卸载带两问向导） |
| `/api/emby/self-update` | 插件自更新 |
| `/api/emby/config` | 读写插件配置（与 Web UI 同源） |
| `/api/emby/libraries` | 媒体库列表 |
| `/api/emby/prepare-output` · `/api/emby/probe-as-emby` | 准备输出目录 / 以 Emby 身份试写 |
| `/api/emby/write-check` | 写权限校验 |
| `/api/emby/sync` · `/api/emby/sync/status` · `/api/emby/sync/stop` | 触发同步 / 进度 / 停止 |
| `/api/sync/purge-preview` | 删除历史数据：只读统计（root / files / bytes / sites / layout / byExt） |
| `/api/sync/purge` | 删除历史数据：`{"confirm":"DELETE"}` |
| `/api/plugin/info` · `/api/plugin/dll` | 插件版本信息 / 下载 DLL |
| `/api/cache/clear` | 清缓存 |

插件侧 `/TvBox/*` 共 14 个路由：

| 端点 | 描述 |
|------|------|
| `/TvBox/Status` | 插件状态 |
| `/TvBox/Sources` | 站点统计（可同步站点数） |
| `/TvBox/Sync` · `/TvBox/Stop` | 触发同步 / 停止 |
| `/TvBox/SelfUpdate` | 插件自更新 |
| `/TvBox/ProbeWrite` | 输出目录试写 |
| `/TvBox/Config` | 读写插件配置 |
| `/TvBox/ConfigBoot` · `/TvBox/ConfigPublic` | 配置页引导脚本 / 公开字段（不含令牌） |
| `/TvBox/Kick` · `/TvBox/Live` | 唤醒 / 直播 M3U |
| `/TvBox/Probe` | 播放地址探测 |
| `/TvBox/PushItem` | 单条推送（搜索页调用） |

## 已知限制

- **dex 内的 spider 类**：TVBox 配置里一部分源把 spider 打进 Android dex，纯 JVM 跑不起来，这类源在站点页显示不可用
- **需要登录的源**（B 站系等）要自备 cookie；**需要外部解析器**的源要自备解析接口模板
- **网盘类源**（继承 `Pan` 基类）没有账号拿不到直链，默认在解析阶段排除
- **源站随时失效**：第三方内容，接口一改就可能播不了，本项目只能保证「探测 → 不写入播不了的条目」

## 版本历史

### v0.1.1 (2026-10-23)

- 首个对外版本：点播源采集 → 站点探测 → 跨站搜索 → 一键推送 → 同步进媒体库
- 输出按站点分目录：`<输出目录>/<站点>/movies|tv/…`（站点层在分类目录外侧，Emby 才好认剧名）
- 新增「删除历史数据」：先统计再确认（需输入 `DELETE`），只删 9 类扩展名，站点根目录与符号链接一律不动
- 插件安装 / 自更新后自动重启 Emby，前端显示重启进度弹窗（成功自动关、失败保留原因、刷新可续接）
- 配置 32 项在 Web UI / 插件页 / API 三入口完全对齐（此前有 12 项只能在插件页改）
- 「全部加入同步」只认已确认可播的站点，无可播时拒绝提交并提示先探测
- 修复：搜索页推送把片名当成站点目录名
- 修复：部署脚本指向不存在的 Emby 应用，插件此前从未真正更新到运行中的 Emby
- 修复：`autoDetectOutputRoot()` 正则多转义导致函数整体不可用

## 维护者

- 作者：[@再见一零一二](https://gitee.com/wyf1015)

---

> 如果这个项目对您有帮助，欢迎赞助支持 ❤️
