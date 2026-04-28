# FnClearup - 清理精灵

> 扫描 FnOS 系统所有 vol 目录，找出已安装应用但残留目录文件的孤立应用。

![FnOS 清理精灵](app/ui/ICON_256.png)

## 功能特性

- **自动扫描** — 动态发现系统中所有 vol 卷（`/vol1` ~ `/vol10`，并自动识别 `@app*` 目录）
- **孤立目录检测** — 交叉比对已安装应用列表与实际文件目录，精准识别残留孤立目录
- **多选删除** — 支持批量勾选，一键清理，附确认弹窗与路径预览
- **Token 鉴权** — 基于 FnOS 配置文件的 Bearer Token 安全机制
- **明暗主题** — 支持手动切换与系统主题跟随（`prefers-color-scheme`）
- **响应式布局** — 适配桌面端与移动端
- **无障碍支持** — 颜色对比度符合 WCAG AA 标准，键盘可聚焦，ARIA 属性完善

## 界面预览

```
┌─────────────────────────────────────┐
│  FnOS 孤立应用清理精灵               │
│  扫描 FnOS 系统所有卷，清理残留目录   │
├─────────────────────────────────────┤
│  📦 已安装应用     📂 孤立目录    💾 受影响卷 │
│       11             12              3        │
├─────────────────────────────────────┤
│  📋 孤立目录（已安装应用不含这些目录） │
│  ☑ app_A  /vol1/app_A   🔵 运行中  │
│  ☑ app_B  /vol2/app_B   🟠 已停止  │
│  [开始扫描]  [🗑 删除选中]           │
└─────────────────────────────────────┘
```

## 快速开始

### 安装

将打包生成的 `FnClearup.fpk` 文件上传至 FnOS 应用中心安装。

### 构建

```bash
cd /app/dist/data/fnclearup
/tmp/fnpack build .
# 输出: FnClearup.fpk
```

### 手动运行（开发调试）

```bash
cd /app/dist/data/fnclearup
pip install -r cmd/fnclearup/requirements.txt
python3 cmd/fnclearup/src/app.py
# 访问 http://<your-ip>:19811/ui
```

## 项目结构

```
fnclearup/
├── README.md
├── manifest.txt              # FnOS 元信息（版本、端口、权限等）
├── cmd/
│   ├── main                   # 应用入口脚本
│   ├── config_init/           # 安装初始化配置
│   ├── config_callback/       # 配置回调
│   ├── install_init/          # 安装初始化
│   ├── install_callback/      # 安装回调
│   ├── upgrade_init/          # 升级初始化
│   ├── upgrade_callback/      # 升级回调
│   ├── uninstall_init/        # 卸载初始化
│   └── fnclearup/
│       ├── requirements.txt   # Python 依赖
│       └── src/
│           ├── app.py          # 主应用（Flask 服务）
│           └── templates/
│               └── index.html  # 前端界面
├── app/
│   ├── ui/                    # 应用图标和赞助二维码
│   ├── config/                # 权限和资源配置
│   └── resource
├── config/
│   ├── privilege              # 权限配置
│   └── resource               # 资源配置
├── wizard/
│   └── uninstall              # 卸载向导
└── ICON.PNG / ICON_256.PNG    # 应用图标
```

## 接口说明

| 接口 | 方法 | 说明 |
|------|------|------|
| `/ui` | GET | 前端界面 |
| `/api/scan` | POST | 扫描孤立目录（返回路径列表） |
| `/api/delete` | POST | 删除指定目录 |
| `/api/status` | GET | 获取扫描状态 |
| `/api/installed` | GET | 获取已安装应用列表 |

## 配置说明

- **服务端口**：默认 `19811`，可通过环境变量 `TRIM_SERVICE_PORT` 覆盖
- **Token 鉴权**：通过 FnOS 配置文件读取，发送请求时携带 `Authorization: Bearer <token>` header
- **卷扫描路径**：动态扫描 `/vol1` ~ `/vol10` 及 `/ff/{hostname}/app*` 目录

## 版本历史

### v0.1.4 (2026-04-28)

**新功能**
- 移除 Token 鉴权，直接打开即可使用，无需任何验证

**Bug 修复**
- 后端：移除 `_load_token`、`_save_token`、`_check_auth`、`@app.before_request` 鉴权中间件及 `/api/status`、`/api/setup` 接口
- 前端：移除 `initAuth`、`showLoginModal`、`showSetupModal`、`submitAuth`、`_authHeaders` 等函数及相关 HTML 弹窗

### v0.1.3 (2026-04-28)

**Bug 修复**
- 修复 CSS 语法错误：`body {` 缺少 `}` 导致所有深色主题样式完全失效

**深色主题完善**
- 补充 setup 弹窗、赞助弹窗、二维码图片、关闭按钮等组件的深色主题适配

**性能优化**
- 所有图片压缩至 ≤100KB（图标转 8 位调色板 PNG，赞助二维码转 JPEG）

### v0.1.2 (2026-04-28)

**Bug 修复**
- 首次设置 Token 时强制隐藏赞助弹窗，避免同时显示
- Token 设置成功后正确隐藏 setup 弹窗并跳转扫描
- 修复环境变量名兼容（`TRIM_SERVICE_PORT` / `UI_PORT` 均支持）

### v0.1.1 (2026-04-27)

**前端优化**
- 顶部统计改为 KPI 卡片式布局（图标 + 渐变背景 + 悬浮效果）
- 颜色对比度优化，符合 WCAG AA 标准
- 添加 `:focus-visible` 焦点样式
- 添加系统主题检测（`prefers-color-scheme`）
- 深色模式颜色优化
- 添加 spinner 加载状态
- 添加 `prefers-reduced-motion` 支持

**后端优化**
- 动态发现 vol 卷（`/vol1` ~ `/vol10`），无需写死
- 端口优先读取 `TRIM_SERVICE_PORT` 环境变量
- 新增 Token 鉴权中间件
- 新增 `/api/status` 接口（返回版本、扫描状态）
- 增强 appcenter-cli 解析容错性（适配 Unicode 表格格式）

### v0.1.0 (2026-04-27)

初始版本，基础扫描与删除功能。

## 依赖

- Python 3.12
- Flask
- appcenter-cli（FnOS 系统工具，通过 PATH 调用）

## 开源许可

MIT License

## 维护者

- 作者：[@一零一二](https://gitee.com/wyf1015)
- FnOS 应用市场

---

> 如果这个项目对您有帮助，欢迎赞助支持 ❤️
