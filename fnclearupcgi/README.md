# 清理精灵 CGI

> 扫描 FnOS 所有 vol 目录，找出已卸载应用（含关联系统用户）、已删除网盘挂载、已删除 docker 残余卷的残留目录，一键清理。

![清理精灵 CGI](ICON_256.png)

## 功能特性

- **自动扫描** — 动态发现系统中所有 vol 卷（`/vol1` ~ `/vol10` 及 `@app*` 目录）
- **孤立目录检测** — 交叉比对已安装应用列表与实际文件目录，精准识别残留孤立目录
- **多选删除** — 支持批量勾选，一键清理，附确认弹窗与路径预览
- **明暗主题** — 支持手动切换与系统主题跟随（`prefers-color-scheme`）
- **Tab 切换** — 网盘挂载目录 / data/vol02 未挂载目录独立展示
- **KPI 卡片** — 显示卷总数、已挂载数量、未挂载数量
- **Docker 卷管理** — 查看在用卷/残余卷，一键批量删除
- **响应式布局** — 适配桌面端与移动端
- **轻量架构** — 纯 Bash CGI 实现，无 Python/Flask 依赖

## 技术架构

```
清理精灵 CGI
├── app/ui/www/           # 前端界面（HTML/CSS/JS）
├── app/api.sh            # CGI 入口，路由 /api/* 请求
├── cmd/                  # FnOS 生命周期脚本
└── config/               # 权限与资源配置
```

- **后端**：Bash CGI + FnOS 原生工具（`appcenter-cli`）
- **前端**：原生 HTML/CSS/JS，无框架依赖
- **通信**：HTTP CGI 请求，无需端口监听

## 快速开始

### 安装

将 `fnclearup.fpk` 上传至 FnOS 应用中心安装。

### 构建

```bash
cd /app/dist/data/fnclearup-cgi
/app/dist/data/tool-bin/fnpack build .
# 输出: App.Native.FnClearup.fpk
```

## 接口说明

| 接口 | 方法 | 说明 |
|------|------|------|
| `/cgi/ThirdParty/App.Native.FnClearup/index.cgi/` | GET | 前端界面 |
| `/cgi/ThirdParty/App.Native.FnClearup/index.cgi/api/scan` | POST | 扫描孤立目录 |
| `/cgi/ThirdParty/App.Native.FnClearup/index.cgi/api/delete` | POST | 删除指定目录 |
| `/cgi/ThirdParty/App.Native.FnClearup/index.cgi/api/installed` | GET | 获取已安装应用列表 |
| `/cgi/ThirdParty/App.Native.FnClearup/index.cgi/api/mounts` | GET | 获取已挂载的网盘目录 |
| `/cgi/ThirdParty/App.Native.FnClearup/index.cgi/api/vol02` | GET | 获取 /data/vol02 目录列表 |

## 版本历史

### v0.3.0 (2026-05-01)

- 前端全面优化：无障碍修复（aria-label、role="dialog"）、模态框标注、alert 阻塞修复
- 新增：网盘 Tab 新增"目录总数"KPI 卡片（显示 /vol02 子目录总数）
- 修复：do_volumes 中 orphan_json 缺少 [] 包裹导致前端 "not iterable" 错误
- 修复：静态文件响应缺少 Status:200 OK 导致浏览器解析失败
- 修复：api.sh 第 524/544 行复杂 pattern substitution 导致 bash -n 报语法错误
- 优化：Docker 残余卷批量获取 driver 信息，O(1) 查表替代逐个 inspect
- 优化：网盘挂载变量名修正（binds_json → mounts_json）

### v0.2.0 (2026-04-30)

- 新增 Tab 切换：网盘挂载目录 / data/vol02 未挂载目录独立展示
- 新增 KPI 卡片：显示已挂载数量和未挂载数量
- 主题按钮样式优化：与赞助按钮风格统一
- CSS 修复：`html.dark` 选择器、重复残缺样式块清理
- JS 修复：App 对象重复属性、`switchTab` 添加 `async` 关键字
- 移除 vol02 列表展开/收起按钮，列表始终显示

### v0.1.5 (2026-04-30)

- 前端审计修复: XSS防护(innerHTML转义)、label for属性、WCAG AA对比度优化

### v0.1.4 (2026-04-30)

- 前端优化: CSS变量化、JS命名空间封装、SEO增强、无障碍改进

### v0.1.3 (2026-04-30)

- 拆分js和css到外部文件，提升可维护性

### v0.1.2 (2026-04-30)

- 第一版发布

## 维护者

- 作者：[@一零一二](https://gitee.com/wyf1015)

---

> 如果这个项目对您有帮助，欢迎赞助支持 ❤️
