# TvBox For Emby

把 TVBox 的点播源接进 Emby：**搜索 → 试播 → 一键推送 → 在 Emby 里当普通影视看**。

一个 fnOS 应用（`tvboxhub.fpk`）+ 一个 Emby 插件（`TvBox.dll`），装完在浏览器里点点鼠标就能用，不用碰命令行。

> 当前版本 **v0.1.0**（首个对外版本）。
> 内部迭代期（0.5.x）的逐条开发记录保留在 [docs/CHANGELOG-internal.md](docs/CHANGELOG-internal.md)。

---

## 为什么需要它

TVBox 生态里的源是「苹果 CMS」接口：能搜、能看详情，但没有媒体库，看完就忘，也没法在电视上按海报浏览。

这个项目做的事很简单：**把点播条目写成 `.strm` 文件**。Emby 把 `.strm` 当普通视频文件扫描入库，于是你得到海报墙、分类、继续观看、多端同步播放进度——而播放时真正拉流的还是源站。

多源的情况也照顾到了：每个站点单独一个目录，同一部片在几个站能看就存几份，谁挂了都不影响。

---

## 组成

| 组件 | 位置 | 说明 |
|---|---|---|
| **fnOS 应用** | `fpk-src/` | 生命周期脚本 `cmd/main`、Web UI、**内置裁剪版 JRE（x64 / arm64 各约 46 MB）**，装到 NAS 上开箱即用 |
| **spider-hub** | `spider-hub/` | Java 中转服务：跑 catvod spider、聚合搜索、代理播放、写 strm 的调度口 |
| **Emby 插件** | `TvBox.Plugin/` | 在 Emby 里提供搜索/推送/播放代理，负责补 Referer/UA 与 HLS 分片重写 |
| **spider 实验舱** | `spider-host/` | 验证「TVBox 的 dex 内 spider 能否直接在 JVM 跑」的探针工具，非运行时依赖 |

hub 与插件之间的调用很薄：插件调 hub 的 `/api/*`，hub 通过插件反向更新媒体库。

---

## 环境要求

- fnOS（含应用中心）
- Emby Server **4.9 / 4.10**（编译参考程序集同时带了这两版）
- 磁盘：输出目录要放 `.strm` + `.nfo` + 海报，几千条量级约几十 MB
- 网络：能访问你选的源站；源站大多在公网

---

## 安装

1. 在 fnOS 应用中心上传 `tvboxhub.fpk` 安装
2. 打开应用 → 顶栏标题卡片右侧有 **💖 赞助 / 💬 联系 / ☾ 主题**，左侧 ☰ 收起展开侧边栏
3. 首次进入会自动弹出**使用向导**；走完或跳过都可以在「使用说明」里重看

`.fpk` 自带 JRE，不需要系统装 Java；x64 / arm64 自动选。

---

## 首次使用（5 步）

UI 的「使用说明」页有同样的清单，并且会**自动检测每一步是否完成**：

1. **填点播源** —— 配置页填 TVBox 配置地址（单仓 JSON / 多仓 urls / base64 / 本机绝对路径都行）
2. **选站点** —— 站点页勾选能用的源。建议先勾 3–5 个 ✅ 的试水
3. **设输出目录** —— 默认在 Emby 数据目录下的 `tvbox/`，也可指到自己的媒体盘
4. **装 Emby 插件** —— Emby 插件页一键安装，装完自动重启 Emby（有进度弹窗，恢复后自动关闭）
5. **跑同步** —— 同步页点开始；跑完在 Emby 里加媒体库并扫描

> 站点页的「**把可播站点全部加入同步**」只加**已确认可播**的站点。
> 「有列表」不等于「能播」——有些源的播放地址返回的是解析接口 JSON。没验证过的站会提示你先点「批量探测当前」。

---

## Emby 媒体库怎么加

同步产物按站点分目录：

```
<输出目录>/<站点名>/movies/<片名 (年份)>/<片名 (年份)>.strm
<输出目录>/<站点名>/tv/<剧名 (年份)>/Season 01/<剧名 (年份)> - S01E01 - <集名>.strm
```

所以媒体库也按站点建：

| 库类型 | 路径 |
|---|---|
| 电影 | `<输出目录>/<站点>/movies` |
| 电视剧 | `<输出目录>/<站点>/tv` |

站点多的话，也可以把 `<输出目录>/<站点>` 加成「**混合内容**」库，一个站点一个库。

命名遵循 Emby 规范：片名带 `(年份)`、集文件名 `剧名 - SxxEyy - 集名`、季目录 `Season NN`。集号优先从源站集名解析（`第12集` / `EP12` / `S02E05` / 纯数字），解析不到才按顺序补。

---

## 播放是怎么走通的

`.strm` 里写的不是源站直链，而是插件的代理地址：

```
http://<你的Emby>/TvBox/Stream?...   →  插件补 Referer / User-Agent  →  m3u8 分片地址重写  →  源站
```

这样防盗链的源也能播，同时播放进度、转码、多端同步都还是 Emby 原生的。

三种流模式（配置页「播放流模式」）：

| 模式 | 行为 | 适用 |
|---|---|---|
| `proxy`（默认） | 全部经 NAS 中转 | 需要补 Referer/UA、需要改写 HLS 分片 |
| `redirect` | 直接 302 到源站 | 省带宽；但防盗链的源会播不了 |
| `auto` | 按「允许 302 的域名」决定，m3u8 仍走代理 | 混合场景 |

概览页的「配置检测」会单独把 `redirect` 标红提醒——这是最容易配错、又最难自查的一项。

---

## 配置项（32 项）

| 分组 | 关键项 |
|---|---|
| **采集源** | `ConfigUrl`、`SiteNameFilter`、`SiteWhitelist`/`SiteBlacklist`、`ExcludeCloudDrive` |
| **同步范围** | `MaxPagesPerCategory`、`MaxItemsPerRun`、`Concurrency`、`TimeoutSeconds`、`RequestDelayMs` |
| **输出** | `OutputRoot`、`WriteNfo`、`DownloadPoster`、`PosterMaxMb` |
| **探测** | `VerifyPlayUrl`、`VerifyMaxEpisodes`、`DetailBatchSize`、`ResolveIndirectUrl` |
| **播放** | `StreamMode`、`RedirectHosts`、`Referer`、`UserAgent`、`UseProxy`、`AllowPrivateHosts` |
| **清理** | `CleanupOrphans`、`MaxDeletePerRun`（单次删除上限，默认 300） |
| **定时** | `EnableScheduledSync`、`SyncIntervalHours` |
| **增量** | `IncrementalSync`、`IncrementalHours` |

同一份配置三个入口（Web UI / 插件页 / API）字段完全对齐，改哪个都落到同一份 `TvBox.xml`。

---

## 同步与清理的安全边界

- **只写不删**：正常情况下同步只写 `.strm` / `.nfo` / 海报
- **孤儿清理有清单**：「我们写过什么」记在 `<输出目录>/.tvbox-manifest.json`，只删清单里出现过、这次又不需要的文件；别人放进去的文件不碰
- **单次删除上限**：一次性要删超过 `MaxDeletePerRun`（默认 300）时直接停手并报错——防止命名规则一改就把整库删空
- **删除历史数据**（同步页）：先统计再确认，必须输入 `DELETE`；只进 `<站点>/movies`、`<站点>/tv` 两个子树，只删 `.strm .nfo .srt .ass .vtt .jpg .jpeg .png .webp .tbn`，不跟随也不删符号链接

---

## Web UI

侧边栏 8 个页面：

| 页面 | 用途 |
|---|---|
| 概览 | KPI + 10 项配置检测 |
| 使用说明 | 首次向导 + 进度清单 + 工作原理 + 常见问题 |
| 站点 | 站点列表、批量探测、加入/移出同步，「可播」单独一列 |
| 搜索 | 跨站点搜索 → 详情 → 试播 → 推送 |
| 同步 | 触发/停止同步、进度、同步范围、**删除历史数据** |
| 配置 | 上面那 32 项 |
| Emby 插件 | 检测 Emby、安装/卸载/自更新插件（带重启进度弹窗） |
| 操作记录 | 每次安装/同步/清理的审计记录 |

移动端：侧边栏默认收起，点 ☰ 打开覆盖式抽屉；标题卡片与动作按钮始终同一行不折。

---

## HTTP API 概览

hub 44 个路由（完整列表见 `spider-hub/src/hub/Hub.java`）：

| 分组 | 代表接口 |
|---|---|
| 站点/配置 | `/api/sites`、`/api/sites/sync`、`/api/sync-sites`、`/api/config`、`/api/save`、`/api/reload`、`/api/prefs` |
| 内容 | `/api/home`、`/api/category`、`/api/detail`、`/api/search`、`/api/search-all`、`/api/probe`、`/api/netdisk` |
| 播放 | `/api/stream`、`/api/stream/seg`、`/api/play-resolve`、`/api/go` |
| Emby | `/api/emby/detect`、`/api/emby/install`、`/api/emby/status`、`/api/emby/sync`、`/api/emby/self-update` |
| 清理 | `/api/sync/purge-preview`、`/api/sync/purge` |
| 其它 | `/api/status`、`/api/history`、`/api/push`、`/api/plugin/info` |

插件 14 个路由（`/TvBox/*`）：`Status`、`Sources`、`Sync`、`Stop`、`SelfUpdate`、`ProbeWrite`、`Config`、`ConfigBoot`、`ConfigPublic`、`Probe`、`PushItem`、`Live`、`Kick`。

**安全**：TCP 只绑 `127.0.0.1`（fnOS 网关走 Unix socket 进来）；写端点要求令牌或同源证明；破坏性端点（安装/卸载/保存/清缓存/删文件）单独列在白名单里强制校验。

---

## 从源码构建

```bash
# 1) Emby 插件（首次会自动下载参考程序集 deb 并解包，不安装）
./build.sh                       # 产物：fpk-src/ui/plugin/TvBox.dll

# 2) spider-hub（Java）
cd spider-hub && ./build.sh      # 产物：hub.jar → fpk-src/ui/hub.jar

# 3) 打包 fpk（在 fpk-src 目录，manifest 所在处）
cd ../fpk-src && fnpack build .
```

依赖：`.NET SDK`（编译插件用 build.sh 里的 `DOTNET` 变量指定）、`JDK`（编译 hub）、`fnpack`、`dpkg-deb`（首次拉参考程序集）。

---

## 已知限制

- **dex 内的 spider 类**：TVBox 配置里一部分源把 spider 打成 Android dex，纯 JVM 跑不起来（实验数据见 `spider-host/`），这类源在站点页会显示不可用
- **需要登录的源**（B 站系等）要自备 cookie；**需要外部解析器**的源要自备解析接口模板
- 网盘类源（继承 `Pan` 基类）没有账号拿不到直链，默认在解析阶段排除
- 源站随时可能失效、改接口——这是第三方内容，不是本项目能修的

---

## 许可与致谢

- 本项目代码：仅供学习与自用，请遵守你所在地区的法律法规与源站条款
- catvod spider 及相关 jar 版权归原作者，本项目只做运行时接入，不修改其逻辑
- 感谢 Emby 社区与 TVBox 生态的公开文档
