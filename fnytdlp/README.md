# fnm3u8dl - 专业视频下载工具

fnOS 系统下的 m3u8/HLS/DASH/MSS 视频下载器，零依赖、纯 Node.js 实现。

## 特性

- **多格式支持**：m3u8/HLS、DASH/MPD、MSS/ISM
- **加密下载**：AES-128 / SAMPLE-AES 解密
- **自动合并**：分片下载完成后自动合并为完整视频
- **直播录制**：m3u8 直播流自动检测 + 持续录制，支持实时合并
- **现代 UI**：自适应桌面/移动端、KPI 卡片、批量操作、搜索、键盘快捷键

## 安装

1. 在 fnOS 应用中心上传 `fnm3u8dl.fpk` 包
2. 启用应用，浏览器访问 fnOS 桌面图标
3. 首次打开会进入"下载设置"页，配置下载路径

## 快速开始

1. 点击 **+** 按钮
2. 输入 m3u8 地址（如 `http://example.com/video.m3u8`）
3. 点击"添加任务"
4. 如果是直播流，会弹"录制设置"对话框
5. 任务会自动下载并在完成时合并

## 设置

⚙️ 图标打开设置弹窗：
- **基本**：下载路径、并发数、超时、重试、User-Agent
- **下载**：线程数、HTTP 超时
- **合并**：合并格式、字幕处理
- **网络**：HTTP 代理、Referer
- **高级**：ffmpeg 路径、检查分片数

## 版本历史

### 0.8.1 (2026-07-28)

**格式修复 + Code Review 重构**

- **修复: 输出格式始终为 .ts** — `muxFormat='auto'` 时等价于 `mp4`，不再跳过混流。默认输出改为 `mp4`
- **修复: ffmpeg 只映射第一路流** — `-map 0:0` → `-map 0`，保留所有视频/音频/字幕流
- **修复: 混流后 `finalSize` 仍读 .ts** — 改为读取混流后文件大小，日志正确显示最终文件路径
- **修复: 混流后 .ts 源文件残留** — 成功混流后自动删除 .ts 源文件
- **修复: yt-dlp 硬编码 .mp4** — 动态使用配置的 `muxFormat` 扩展名
- **修复: ffmpeg/mkvmerge 无超时** — 加 `timeout: 300000` 防止进程泄漏
- **Code Review 发现并修复 5 个严重问题** (大小/日志/扩展名/残留/超时)

### 0.8.0 (2026-06-16)

- **视觉重构: 蓝紫色系主题** — 整体色板从绿色切换为蓝紫色系 (hsl(220, 60%, 55%))，header 蓝-青-靛渐变
- **交互动效增强** — 按钮按压缩放 (0.97)、hover 上浮、弹窗 backdrop blur 渐入 + scale 入场、Toast spring 弹性动画
- **任务卡片状态视觉** — 左侧状态色条 (蓝/绿/红/黄)、骨架屏 loading
- **CSS 变量补齐** — 新增 `--radius-md`、`--transition-normal`、`--text-subtle`，统一圆角层级 (xs/sm/md/lg)
- **新增: 网页嗅探 m3u8** — 输入网页 URL（B站、YouTube 等），服务端抓取页面 HTML 正则搜索媒体链接，B站/YouTube 走 yt-dlp --dump-json 回退
- **新增: yt-dlp 下载模式** — 嗅探/手动添加任务可选择 yt-dlp 下载，支持格式选择、实时进度、cookie 传入
- **新增: Cookie 管理模块** — 上传/管理 Netscape 格式 cookies.txt，添加任务时按域名匹配或手动选择
- **安全加固: Cookie 上传大小限制 (1MB)、路径穿越校验增强**
- **测试: 新增 Cookie API + Sniff + yt-dlp + inline onclick 测试覆盖**
- **版本: 0.8.0 (视觉重构 + 嗅探 + yt-dlp + Cookie 四大模块)**

### 0.7.1 (2026-06-10)

- **前端代码审查修复 12 项**
  - **P0-1** 浏览目录 onclick 单引号 XSS：改用 `data-path` + `this.dataset` 避免函数参数字符串注入
  - **P1-1** 批量添加传 options：合并 `settings.maxConnections/timeout/retryCount` 全局默认值
  - **P1-2** `deleteSchedule` 加 `window.confirm()` 二次确认
  - **P1-3** 内联转义函数统一：3 处 `s.replace(/[&<>"]/g, ...)` 合并为 `escHtml()`
  - **P1-4** 定时状态文本提取为 `SCHEDULE_STATUS_META` 常量（与 `STATUS_META` 分离）
  - **P2-1** URL 截断改用 `[...str].slice(0,50).join('')` 防 surrogate pair 切断
  - **P2-2** `loadTasks` 完成后启动 `setInterval(pollTasks)`，消除首次加载竞态
  - **P2-3** 空状态 inline style 替换为 `.empty-state` 类
  - **P2-4** server.js 浏览路径添加系统目录保护（`/proc` `/sys` `/dev` `/etc` `/boot` 拒绝）
  - **P2-5** 定时列表时间显示统一用 `formatTime()` 辅助函数
  - **P2-6** 合并 `escapeHtml` + `escHtml` 双函数为单一 `escHtml`（正则实现）
  - **P3-1** 主题切换 CSS transition 平滑过渡（背景/文字 0.3s）
- **测试**：语法 + 功能测试通过，git commit 2f6d4ad + 2bfbb54

### 0.7.0 (2026-06-09)

- **下载路径浏览对话框**（参照 fnm3u8dl → fnytdlp 同款）
  - 服务端新增 `GET /api/browse?path=...` 端点（不限制白名单，用户可任意浏览后选目录）
  - 设置面板下载路径输入框右侧加 `📂 浏览` 按钮
  - 弹窗显示当前路径 + 上级目录 + 子目录列表 + 选择当前目录
  - 选择后通过 `POST /api/config` 保存，触发 `registerPathPrefix` 提前注册
- **页脚版权信息**（参照 fnytdlp）
  - 页面底部居中显示 `fnm3u8dl v0.7.0 · © 2026 一零一二`
  - server.js 新增 `VERSION` 常量，`/api/health` 暴露版本号给前端动态渲染
- **时钟 / 速度 KPI 修复**
  - 时钟 `--:--:--` 不显示：`sparkline.js` 使用 ESM export，script 标签缺 `type="module"` 导致脚本不执行 → 修复
  - 实时速度 KPI 数值换行：字号 1.1→0.95rem + `white-space: nowrap` + ellipsis
- **错误处理强化**
  - `r.pipeThrough is not a function` 500 bug 修复（ArrayBuffer 没 pipeThrough API）
  - 浏览端点拒绝未注册路径导致无法选目录 → 改为不限制白名单
- **测试**：单元测试 132/132 全过；端到端 4 场景浏览验证通过

### 0.5.0 (2026-06-07)

- **直播录制功能完整补齐**（对照 C# N_m3u8DL-RE 10 个直播参数）
  - `livePipeMux` 增量追加模式（用 `appendFileSync` 模拟 ffmpeg pipe mux）
  - 增强直播检测：URL 关键词 + 短 playlist 启发式 + 缺失 ENDLIST
  - 添加任务时自动检测直播，弹"录制设置"对话框
  - 修复 5 个直播录制 bug：参数丢失 / 录制只下 1 分片 / 实时合并误改 status / addTask 双层包装 / 直播状态显示
- **设置保存修复**：`isSafePath` 不再拒绝新下载路径，前端检查响应状态
- **UI 移动端适配**：KPI 2 列布局 / 弹窗全屏 / 480px 断点 / 按钮紧凑
- **8 项视觉重构**：Header / KPI / Toolbar / Task Card / Badge / Empty / Batch / Sidebar / Sparkline
- **前端 6 个新 UI**：autoSelect / subOnly / baseUrl / allowHlsMultiExtMap / muxImport / taskStartAt
- **URL 自动解码**：用户从聊天复制 percent-encoded URL 自动识别
- **15 个新 TDD 测试**：live_detect / live_pipe_mux / live_status / addTask body structure 等
- **总数 188+ 测试通过**

### 0.6.1 (2026-06-09)

- **安全加固**（前端全面审查 8 项修复）
  - file:// SSRF 白名单校验
  - IPv4 映射 IPv6 内网地址绕过修复
  - fetch 超时控制（30s）
  - 错误信息路径泄露过滤
  - url-resolver 流式 body 读取限 1MB
  - AbortSignal.any try/catch 兜底
  - 默认日志级别从 info 改为 warn，避免 fnOS info.log 膨胀
- **代码质量**
  - batchDelete 异步 await 修复
  - addTask 重复 ID 去重
  - CORS_ORIGINS 空值时同源回退
  - textarea CSS 独立类（textarea-mono）
  - sparkline 颜色格式扩展（hsl/rgba）
  - 删除死代码 sampleData
  - 删除多余 CSS 大括号
- **测试**：270/270 通过，0 失败

## 项目结构

```
app/ui/
├── server.js              # HTTP server + API routes
├── main.js                # 前端逻辑
├── index.html             # HTML
├── live/                  # 直播录制
│   └── http-live-record-manager.js
├── parser/                # m3u8/MPD 解析
├── downloader/            # 下载器
├── merge/                 # 合并工具
└── util/                  # 工具函数
cmd/                      # fpk 命令行入口
config/                   # fpk 配置
manifest                  # fpk manifest
```

## 开发

```bash
# 运行测试
npm test

# 打包 fpk
fnpack build .
```

## 许可

MIT

## 更新日志

### v0.8.1 (2026-07-04)
- **修复: 非原子写入** — saveTask restart / cookie meta 改为 tmp+renameSync 原子模式，崩溃不丢数据
- **修复: EncryptInfo 共享引用** — DASH 解析所有分片独立 clone，避免 KID 污染
- **修复: Pipe hang** — merge-util 加 dest error/close 检测，文件流提前关闭不再挂死
- **修复: 广告过滤双 pop** — hls-extractor YK 广告过滤 `else-if` 防重复弹出正常分片

### v0.8.0 (2026-06-16)
- **视觉重构**: 蓝紫色系主题 + 交互动效 + 任务卡状态色条 + 骨架屏
- **新增**: 网页嗅探 m3u8 + yt-dlp 下载模式 + Cookie 管理模块
- **安全**: Cookie 上传 1MB 限制 + 路径穿越校验
- **测试**: 493+ pass，inline onclick 全量覆盖

### v0.7.1 (2026-06-10)
- 修复: 浏览目录 onclick 单引号 XSS (改用 data-path + this.dataset)
- 修复: 批量添加不传 options (合并 settings 默认值)
- 修复: deleteSchedule 无确认 (加 window.confirm)
- 修复: 内联 HTML 转义函数重复 3 次 (统一 escHtml)
- 修复: 定时状态文本硬编码 (提取 SCHEDULE_STATUS_META)
- 修复: substring(0,50) 切断 surrogate pair ([...str].slice)
- 修复: loadTasks/pollTasks 竞态 (then 后启动轮询)
- 修复: 空状态 inline style (.empty-state 类)
- 修复: server 浏览无系统目录保护 (/proc /sys /dev /etc /boot 拒绝)
- 修复: 定时列表时间 toLocaleString (统一 formatTime)
- 修复: escapeHtml/escHtml 双函数并存 (合并为 escHtml)
- 优化: 主题切换 CSS transition (背景/文字 0.3s)

### v0.7.0 (2026-06-09)
- 新增: 下载路径浏览对话框 (📂 按钮 + 弹窗选择)
- 新增: 页脚版权信息 (动态版本号 + 自动年份)
- 修复: 时钟 --:--:-- 不显示 (sparkline.js ESM 模块加载)
- 修复: 实时速度 KPI 换行 (字号 + nowrap)
- 修复: r.pipeThrough TypeError 500 (ArrayBuffer API 误用)
- 优化: 浏览端点不限制白名单 (可任意浏览后选目录)
- 测试 132/132 通过

### v0.6.1 (2026-06-09)
- **安全加固**: SSRF/IPv6绕过/超时控制/路径泄露/流式body限流/日志级别warn
- **代码质量**: batchDelete await/去重/CORS回退/CSS优化/死代码清理
- **测试** 270/270 通过，0 失败

### v0.6.0 (2026-06-08)
- 新增: 定时录制列表编辑和真删除功能
- 修复: 工具栏 ghost 按钮背景色与其他按钮一致，边框样式统一
- 优化: 设置/搜索按钮移入工具栏，顶部 header 精简对齐

### v0.5.1 (2026-06-08)
- 修复: formatDuration 秒→毫秒单位不匹配 (录制时长显示错误)
- 修复: 0 秒录制显示为 '-' 而非 '0秒'
- P0-3: file:// SSRF 白名单校验
- P1-1: IPv4映射IPv6内网地址绕过修复
- P1-5: batchDelete 异步 await 修复
- P1-6: addTask 重复 ID 去重
- P1-7: fetch 超时控制 (30s)
- P1-9: CORS_ORIGINS 空值时同源回退
- 测试 432/435 通过 (3 跳过为网络相关)
