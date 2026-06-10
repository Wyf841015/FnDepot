# fnytdlp 视频下载器 (v0.2.5)

> 集成 yt-dlp (1872+ 站点) 的 fnOS 视频下载工具，零依赖、纯 Node.js。

## 简介

支持 YouTube / B 站 / 抖音 / 微博 / X / 西瓜 / 推特 / Twitch / Vimeo 等任意 yt-dlp 支持的站点。

**核心特性：**
- yt-dlp 完整参数：格式选择、字幕下载、SponsorBlock 广告跳过、Cookie 认证、代理
- 自适应 x86_64 + aarch64 双 binary（启动时按 `process.arch` 自动选）
- 统一网关模式：Unix socket + HTTP 双端口
- 4 张 KPI 卡片 + Sparkline 实时趋势
- SSE 实时进度推送
- 任务详情弹窗（13 字段）
- 下载路径浏览对话框
- 多网站 Cookie 任务级透传
- 5 主题 + 响应式布局（桌面/移动端）

**安全：** SSRF 防护 / 路径白名单 / Cookie 加密存储 / 30s fetch 超时 / 错误脱敏

**系统依赖：** Node.js 20+ (fnOS 自带) / Python 3.7+ (yt-dlp zipimport) / ffmpeg 7.1.3 (fnOS 系统内置)

## 安装

1. 在飞牛 NAS 应用中心 → 手动安装 `fnytdlp.fpk`
2. 启用应用，浏览器访问桌面图标
3. 首次进入"设置"，配置下载路径

## 维护者

- 作者：[@再见一零一二](https://gitee.com/wyf1015)
- GitHub：[@Wyf841015](https://github.com/Wyf841015)
- Gitee：[@wyf1015](https://gitee.com/wyf1015)
- 第三方依赖：[yt-dlp](https://github.com/yt-dlp/yt-dlp) (Unlicense 公共领域)
