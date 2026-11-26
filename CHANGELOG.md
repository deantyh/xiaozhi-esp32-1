# 修改日志 / Changelog

本文件记录基于官方 `78/xiaozhi-esp32` 的本地硬件适配分支的变更。
格式参考 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)。

## [v2.6.0-s3cam240x320-dl.1] - 2026-11-26

### 新增 / Added
- 新增构建变体 `bread-compact-wifi-s3cam-240x320-dl`，面向面包板 ESP32-S3-CAM + ST7789 240×320 LCD + OV 摄像头。
  - 固定 `CONFIG_LCD_ST7789_240X320=y`，确保屏幕参数与当前硬件一致。
  - 开启 `camera_hmirror=true`，与之前 v2.5.2 固件的摄像头画面方向保持一致。
- 恢复电池供电能力（官方 v2.6.0 从该板型中移除了相关代码）：
  - 从 v2.5.2 移植 `power_manager.h`，通过 ADC2 读取电池电量、GPIO3 检测充电状态。
  - 新增 Kconfig 选项 `CONFIG_BREAD_COMPACT_WIFI_CAM_POWER_MANAGEMENT`，默认关闭，仅在定制变体中开启。
  - 空闲 60 秒进入低功耗显示、36000 秒（10 小时）请求深度睡眠；外接供电时自动关闭休眠。
  - 实现 `GetBatteryLevel()`，使天气/电量查询和 MCP `self.get_device_status` 能读到真实电量。

### 修改 / Changed
- `main/boards/bread-compact-wifi-s3cam/compact_wifi_board_s3cam.cc`
  - 在保留官方摄像头、按键、显示初始化的前提下，按编译开关接入电源管理。
  - 深度睡眠前拉低并保持板载 LED 所在 RTC GPIO，降低待机功耗。

### 说明 / Notes
- 基线：官方 tag `v2.6.0`（commit `4de00ae`），上游仓库 `https://github.com/78/xiaozhi-esp32`。
- 编译要求：ESP-IDF `release/v6.1` 分支。
- 屏幕：ST7789 SPI，240×320，SPI mode 0，RGB 顺序，IPS 反色开启。
- 未改动官方联网、语音、MCP、OTA 逻辑；未内置任何云端账号或激活信息。
