# 项目上下文

## 项目概述
- **DuckyClaw**: 运行在边缘设备上的硬件 AI Agent，通过 IM 通道（Telegram / Discord / Feishu）以自然语言与用户交互
- **技术栈**: C（TuyaOpen SDK）、CMake、Kconfig，跨平台部署（Tuya T5AI / ESP32-S3 / Raspberry Pi / Linux）

## 架构概览
- **消息总线**: 统一使用 `sys_bus`（`include/sys_bus.h`），`sys_msg_t` 结构体
- **Agent 循环**: `agent_loop_task` 外循环等待入站消息，内循环最多 10 轮工具迭代
- **事件驱动初始化**: 多数模块通过 `tal_event_subscribe(EVENT_MQTT_CONNECTED, ...)` 延迟初始化
- **持久化记忆**: MEMORY.md / 日记 / SOUL.md / USER.md，通过 `claw_f*` 宏读写 SD 或 flash

## 约定与规范
- 详见 `AGENTS.md` 中的 9 条 Guideline
- 新增代码遵循 TuyaOpen C 代码风格（doxygen 注释、OPERATE_RET 返回值、tal_* API）

## 当前状态
- 核心功能（Agent Loop、MCP 工具、多通道 IM、持久化记忆）已完成
- 待办：音频 ASR 输入、更多 CLI 设置项（见 TODOs.md）
