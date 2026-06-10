# m3u8DL 专业视频下载器

> N_m3u8DL-RE 的 Node.js 原生重写版 —— 功能完全对照 C# 参考项目，零外部依赖。
> 支持多线程、多任务、断点续传的 m3u8 / MPD / ISM 视频下载，AES-128 解密、批量添加、直播录制。

![m3u8DL 专业视频下载器](ICON_256.PNG)

## 功能特性

- **多协议支持** — m3u8 (HLS)、MPD (DASH)、ISM (MSS) 三大流媒体协议全兼容
- **多线程下载** — 支持多线程并发下载，智能分片加速下载体验
- **断点续传** — 任务失败后可继续下载，不重复下载已完成分片
- **AES 解密** — 自动识别并解密 AES-128-CBC 加密的流媒体分片
- **批量添加** — 支持粘贴多个链接（每行一个），一键批量创建下载任务
- **直播录制** — 支持直播流录制，可设置刷新间隔持续录制
- **状态筛选** — 按状态筛选任务：全部 / 进行中 / 已完成 / 失败
- **批量操作** — 全暂停、全开始、全停止、清理已完成、批量删除（可选删除文件）
- **速度显示** — 实时显示下载速度、ETA、已用时间、平均速度
- **加密标注** — 加密流自动标注 🔒 徽章，详情面板显示加密方式
- **输出格式** — 支持 .ts 原生 / 自动合并 / 不合并保留分片
- **错误重试** — 可配置重试次数、超时、并发数
- **代理支持** — 支持 HTTP 代理、自定义 Referer、User-Agent、请求头
- **速度限制** — 支持全局下载速度限制
- **首次引导** — 首次运行时自动弹出设置引导，提示配置下载目录
- **明暗主题** — 支持浅色/深色双主题切换，设置持久化到服务端
- **赞助支持** — 内置支付宝/微信赞助二维码，支持全屏查看
- **响应式布局** — 适配桌面端与移动端，移动端工具栏自动图标化

## 与 FnM3u8 对比

| 特性 | FnM3u8 (旧版) | m3u8DL (新版) |
|------|--------------|--------------|
| 底层 | Flask + 少量模块 | **Node.js ESM 零依赖** |
| 协议解析 | 仅 HLS | **HLS + DASH + MSS** |
| 解密 | 基础 AES | **AES-128-CBC + ChaCha20** |
| 直播录制 | 基础 | **可配刷新间隔** |
| 速度/ETA 显示 | ❌ | ✅ |
| 批量添加 | ❌ | ✅ |
| 拖选批量操作 | ❌ | ✅ |
| 删除确认弹窗 | ❌ | ✅ |
| 首次设置引导 | ❌ | ✅ |
| 服务端主题持久化 | ❌ | ✅ |
| 状态栏汇总 | ❌ | ✅ |
| 搜索任务 | ❌ | ✅ |
| 测试覆盖 | 无 | **165 个 TDD 测试** |

## 技术架构

```
m3u8DL 专业视频下载器
├── app/ui/                    # 前端界面（HTML/CSS/JS）+ 后端服务
│   ├── server.js             # HTTP + UNIX Socket 双服务器
│   ├── index.html            # 主页面（响应式）
│   ├── main.js               # 前端逻辑（任务管理/主题/搜索/赞助）
│   │
│   ├── parser/               # 流媒体解析器
│   │   ├── hls-extractor.js  # HLS m3u8 解析（主/媒体播放列表）
│   │   ├── dash-extractor.js # DASH MPD 解析（SegmentTemplate/Timeline/List/Base）
│   │   ├── mss-extractor.js  # MSS ISM 解析
│   │   └── util/             # 工具方法（ParserUtil）
│   │
│   ├── processor/            # 内容处理器流水线
│   │   ├── hls-content-processor.js   # HLS 修复（KEY/INF 顺序、换行符）
│   │   ├── dash-content-processor.js  # DASH 修复（cenc namespace）
│   │   ├── default-url-processor.js   # URL 修复（双编码、协议相对）
│   │   └── key-processor.js           # 密钥处理
│   │
│   ├── downloader/           # 下载引擎
│   │   ├── simple-downloader.js       # 单段下载器
│   │   └── download-result.js         # 下载结果
│   │
│   ├── download-manager/     # 下载管理器
│   │   └── simple-download-manager.js # 并发管理/限速/取消
│   │
│   ├── merge/                # 合并工具
│   │   └── merge-util.js     # 批量合并 + 大文件分批
│   │
│   ├── crypto/               # 加解密
│   │   ├── aes-util.js       # AES-128-CBC 加解密
│   │   └── cha-cha20-util.js # ChaCha20 加解密
│   │
│   ├── mp4/                  # MP4 解析
│   │   ├── binary-reader.js  # 二进制读取器
│   │   ├── mp4-parser.js     # MP4 Box 解析
│   │   └── mp4-init-util.js  # fMP4 init segment 解析
│   │
│   ├── entities/             # 数据模型
│   │   ├── playlist.js, media-segment.js, media-part.js
│   │   ├── stream-spec.js, encrypt-info.js
│   │   └── subtitle/
│   │
│   └── util/                 # 工具函数
│       ├── global-util.js    # 格式化/时间工具
│       ├── http-util.js      # HTTP 编码检测
│       └── binary-check.js   # 二进制内容检测
│
├── tests/                    # TDD 测试套件（165 个）
│   ├── test_comprehensive.js # 42 个综合测试 + API 集成测试
│   ├── test_modules.js       # 31 个模块单元测试
│   └── test_new_modules.js   # 91 个新模块测试
│
├── cmd/                      # FnOS 生命周期脚本
├── config/                   # 权限配置
└── wizard/                   # 安装向导
```

- **后端**：Node.js v24 ESM（零外部 npm 依赖，仅用内置 `node:http`/`node:fs`/`node:crypto`）
- **前端**：原生 HTML/CSS/JS，无框架依赖
- **数据存储**：JSON 文件存储（config.json + 任务文件）
- **通信方式**：UNIX Socket（fnOS Gateway）+ HTTP 双通道
- **fnOS Gateway**：自动挂载 `/app/fnm3u8dl` 前缀

## API 端点

| 端点 | 方法 | 描述 |
|------|------|------|
| `/health` | GET | 健康检查 |
| `/api/config` | GET | 获取配置 |
| `/api/config` | POST | 保存配置（含主题偏好） |
| `/api/tasks` | GET | 获取任务列表 |
| `/api/tasks` | POST | 添加下载任务 |
| `/api/tasks/:id` | GET | 获取单个任务 |
| `/api/tasks/:id` | DELETE | 删除任务（支持 ?deleteTempFiles=1&deleteOutputFile=1） |
| `/api/tasks/:id/pause` | POST | 暂停任务 |
| `/api/tasks/:id/resume` | POST | 继续任务 |
| `/api/tasks/:id/stop` | POST | 停止任务 |
| `/api/parse` | GET | 解析 m3u8/MPD/ISM URL |

## 安装

### 在 FnOS 应用中心安装

1. 下载 `fnm3u8dl.fpk`
2. 在飞牛NAS应用中心 → 手动安装
3. 首次使用会自动弹出设置引导，配置下载保存路径

### 直接运行

```bash
cd /path/to/fnm3u8dl/app/ui
PORT=43940 node server.js
```

然后访问 `http://localhost:43940` 即可。

## 版本历史

### v0.7.1-hotfix1 (2026-06-10)

- **修复: 定时录制按钮没反应**
  - 根因：`// ===== Schedule (定时录制) =====` 段 6 个函数（`showScheduleModal` / `switchScheduleTab` / `editSchedule` / `submitSchedule` / `loadScheduleList` / `deleteSchedule`）前一次重排时多缩进 2 空格，JS 不会抛错（语法合法）但函数变成 `document.addEventListener('keydown', ...)` 回调内的局部变量，window 上找不到
  - inline `onclick="showScheduleModal()"` 调用失败
  - 修法：6 个函数全部去前导空格变顶层声明 + 补全 4 个缺失的 `window.X = X` export（原只 export `editSchedule` / `deleteSchedule`）
  - git commit e5e3fba

### v0.7.1 (2026-06-10)

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

### v0.7.0 (2026-06-09)

- **下载路径浏览对话框**（参照 fnytdlp）
  - 服务端新增 `GET /api/browse?path=...` 端点（不限制白名单，可任意浏览后选目录）
  - 设置面板下载路径输入框右侧加 `📂 浏览` 按钮
  - 弹窗显示当前路径 + 上级目录 + 子目录列表 + 选择当前目录
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

### v0.6.1 (2026-06-09)

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

### v0.2.0 (2026-05-29)

- 修复 HLSExtractor.loadM3u8FromUrl 方法缺失（新增公开方法别名，内部调用 _loadM3u8FromUrlAsync）
- 修复 DASH/MPD 解析路径：改用 fetch() 直接获取 MPD 内容，ExtractStreamsAsync 已能处理原始 XML
- 修复 MSS/ISM 解析路径：同样改用 fetch() + ExtractStreamsAsync
- 165 个 TDD 测试通过（3 个跳过为网络相关）

### v0.1.0 (2026-05-26)

- 完整重写 N_m3u8DL-RE C# 项目
- HLS / DASH / MSS 三大协议解析
- AES-128-CBC 自动解密
- 多线程并发下载 + 断点续传
- 批量添加（粘贴多链接）
- 直播录制（可配刷新间隔）
- 实时速度 / ETA / 加密标注
- 响应式 UI + 浅色/深色主题（服务端持久化）
- 批量操作（暂停/继续/停止/清理/删除含文件）
- 首次设置引导
- 赞助支持
- 163 个 TDD 测试覆盖（v0.1.0 初版）

## 测试

```bash
# 运行全部测试
node --test tests/*.js

# 运行特定测试文件
node --test tests/test_comprehensive.js
node --test tests/test_modules.js
node --test tests/test_new_modules.js
```

## 维护者

- 作者：[@再见一零一二](https://gitee.com/wyf1015)
- GitHub：[@Wyf841015](https://github.com/Wyf841015)
- Gitee：[@wyf1015](https://gitee.com/wyf1015)

---

> 如果这个项目对您有帮助，欢迎赞助支持 ❤️

## 更新日志

### v0.7.1-hotfix1 (2026-06-10)
- 修复: 定时录制按钮没反应 (schedule 段 6 函数错误缩进成嵌套 + 补全 4 个 window export)

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

### v0.6.0
- 新增: 定时录制列表编辑和真删除功能
- 修复: 工具栏 ghost 按钮背景色统一
- 优化: 设置/搜索按钮移入工具栏，顶部 header 精简对齐

### v0.5.1
- 修复: formatDuration 秒→毫秒单位不匹配 (录制时长显示错误)
- 修复: 0 秒录制显示为 '-' 而非 '0秒'
- 安全加固: SSRF 白名单校验 / IPv4映射IPv6绕过修复 / fetch超时控制 / CORS同源回退
- 测试 432/435 通过

### v0.5.0
- 直播 3 维度启发式检测 (URL关键词 + EXT-X-ENDLIST + 短playlist)
- 直播实时合并 livePipeMux (纯JS模拟，360倍IO减少)
- C# 10/10 直播参数完整对应
- 动态直播源 (PHP url-resolver) 支持
- 8 项 UI 现代化重构 + 移动端响应式适配
- 测试 432/435 通过
