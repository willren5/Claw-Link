# Claw Link

Claw Link 是一个 **OpenClaw Gateway 的移动遥控器**（iOS/Expo），不是本地模型运行端。

它面向自托管用户，支持在局域网、VPS、Tailscale/WireGuard 环境下远程管理网关，覆盖：

- Dashboard（统计与成本）
- Agents & Skills（代理与技能管理）
- Monitor（日志流与系统监控）
- Multi-Agent Chat（多代理会话与离线队列）

## 设计原则

- 纯遥控：所有推理在网关端执行，移动端只做控制与展示
- 低开销：限制缓存大小、限制日志长度、避免无效重渲染
- 抗断网：消息队列本地持久化，重连后增量对账
- 安全优先：Token 仅存 SecureStore，危险操作启用生物认证
- 零遥测：无三方追踪、无分析埋点

## 技术栈

- Expo + React Native + expo-router
- TypeScript strict
- Zustand + MMKV
- Axios（拦截器 + 重试）
- WebSocket（日志/指标）
- SSE（聊天流式）
- Zod（运行时接口校验）

## 主要模块

### 1) Connection Manager

- 首次启动先走权限引导（相机/相册/麦克风），完成后进入登录连接页
- 网关配置管理（最多 5 个）
- `/api/health` + `/api/devices` 连接握手
- 30 秒心跳保活 + 断连提示

### 2) Dashboard

- 实时拉取：请求量、Token、延迟、成本
- 本地成本估算（可配置模型价格）
- 自动刷新（30s/60s/5min）与手动刷新

### 3) Agents & Skills

- 代理列表、启停、重启、强制 Kill
- 日志查看（最近 100 行）
- 危险操作生物认证

### 4) Monitor

- 网关日志 WebSocket 流
- 主机探针指标流（CPU/RAM/IO/网络/GPU）
- 紧急控制（重启网关、清空会话）

### 5) Chat

- 多代理会话
- SSE 流式输出
- 本地消息持久化 + pending 队列
- 断网重连后 hash 对账与增量补齐

## 性能与内存策略

- Tab 页面启用 `freezeOnBlur`，高开销页面 `unmountOnBlur`
- 日志、指标、消息均有上限（ring buffer）
- 聊天流式 token 合并刷新（减少频繁 setState）
- WebSocket/SSE 在组件卸载时统一释放
- 长列表使用 `FlatList` 虚拟化参数

## 安全策略

- API Token 存储在 `expo-secure-store`（Keychain）
- MMKV 仅保存业务状态与缓存，不保存明文 token
- 危险操作统一 `expo-local-authentication`
- 接口数据统一 Zod 校验

## 目录

- `app/`：路由与页面入口
- `src/features/`：按功能拆分模块
- `src/lib/api/`：API client 与 endpoints
- `src/lib/schemas/`：Zod schema
- `src/lib/security/`：认证能力
- `docs/`：说明书与开发日志

## 快速开始（开发）

1. 安装依赖
2. 启动 Expo 开发服务
3. 打开 iOS 模拟器或真机
4. 首次进入 App 先授予权限，再填写网关 Host/Port/Token 登录连接

> 本仓库当前以功能代码为主，若你在新环境初始化项目，请先补齐 Expo 工程标准文件（`package.json`, `app.json` 等）。

## 参考文档

- 使用说明：`docs/MANUAL.md`
- 开发日志：`docs/WORKLOG.md`
