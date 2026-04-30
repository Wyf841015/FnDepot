# 清理精灵 CGI

> 扫描 FnOS 所有 vol 目录，找出已安装应用但残留目录文件的孤立应用，一键清理。

![清理精灵 CGI](ICON_256.png)

## 功能特性

- **自动扫描** — 动态发现系统中所有 vol 卷（`/vol1` ~ `/vol10` 及 `@app*` 目录）
- **孤立目录检测** — 交叉比对已安装应用列表与实际文件目录，精准识别残留孤立目录
- **多选删除** — 支持批量勾选，一键清理，附确认弹窗与路径预览
- **明暗主题** — 支持手动切换与系统主题跟随（`prefers-color-scheme`）
- **响应式布局** — 适配桌面端与移动端
- **轻量架构** — 纯 Bash CGI 实现，无 Python/Flask 依赖

## 技术架构

```
清理精灵 CGI
├── app/ui/www/           # 前端界面（HTML/CSS/JS）
├── app/api.sh             # CGI 入口，路由 /api/* 请求
├── cmd/                   # FnOS 生命周期脚本
└── config/                # 权限与资源配置
```

- **后端**：Bash CGI + FnOS 原生工具（`appcenter-cli`）
- **前端**：原生 HTML/CSS/JS，无框架依赖
- **通信**：HTTP CGI 请求，无需端口监听

## 快速开始

### 安装

将 `fnclearupcgi.fpk` 上传至 FnOS 应用中心安装。

### 构建

```bash
cd /app/dist/data/fnclearup-cgi
/app/dist/data/tool-bin/fnpack build .
# 输出: fnclearupcgi.fpk
```

## 接口说明

| 接口 | 方法 | 说明 |
|------|------|------|
| `/cgi/ThirdParty/App.Native.FnClearup/index.cgi/` | GET | 前端界面 |
| `/cgi/ThirdParty/App.Native.FnClearup/index.cgi/api/scan` | POST | 扫描孤立目录 |
| `/cgi/ThirdParty/App.Native.FnClearup/index.cgi/api/delete` | POST | 删除指定目录 |
| `/cgi/ThirdParty/App.Native.FnClearup/index.cgi/api/installed` | GET | 获取已安装应用列表 |

## 版本历史

### v0.1.2 (2026-04-30)

- 第一版发布

## 维护者

- 作者：[@一零一二](https://gitee.com/wyf1015)

---

> 如果这个项目对您有帮助，欢迎赞助支持 ❤️
