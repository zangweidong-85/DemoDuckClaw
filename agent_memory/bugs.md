# 问题与风险记录

## 已知问题
- T5 平台获取 RSSI 会崩溃（`tkl_wifi_station_get_conn_ap_rssi`），已通过 `#ifndef PLATFORM_T5` 规避

## 风险项
- 内存紧张：ESP32-S3 等平台堆空间有限，大 buffer 需使用 PSRAM（`ENABLE_EXT_RAM`）
- 跨平台条件编译分支增多，需注意回归测试覆盖

## 技术债务
- `message_bus`（`IM/bus/message_bus.h`）为遗留基础设施，待逐步迁移到 `sys_bus`
- 部分模块（`app_im`）仍混用旧类型 `BOOL_T` / `VOID_T`，需在后续维护中逐步替换
