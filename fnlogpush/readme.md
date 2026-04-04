# 日志哨兵 (FNLogPush)

自动监控飞牛 NAS 系统日志和备份状态，实时推送至多渠道。

## v0.6.9 更新内容

### 🆕 新增渠道
- **MeoW 推送渠道** - 支持 MeoW App 消息推送

### 🔧 优化
- 页面初始化时同步加载所有状态，避免显示延迟

## v0.6.8 更新内容

### 🆕 新增事件
- **存储管理类事件**：CreateStorage、MountStorage、UnmountStorage、DeleteStorage、STORAGE_MOUNT_SUCCESS/FAILED、STORAGE_UMOUNT_SUCCESS/FAILED
- **防火墙类事件**：FW_ENABLE、FW_DISABLE、FW_START_SUCCESS/FAILED、FW_STOP_SUCCESS/FAILED

### 🔧 优化
- 新增"存储管理"和"防火墙"事件分类
- 新事件默认添加到订阅列表，避免老用户看到"发现新事件"提示
