# 硬件适配说明（v2.6.0-s3cam240x320-dl.1）

## 已构建固件产物
本次定制固件已通过 GitHub Actions 云端构建成功，产物位于：
- `releases/v2.6.0_bread-compact-wifi-s3cam-240x320-dl_merged.bin`（9.65 MB，单文件合并固件，烧录到 `0x0`）
- `releases/v2.6.0_bread-compact-wifi-s3cam-240x320-dl.zip`（2.52 MB，完整发布包，内含 `merged-binary.bin`）
- 同样存放在工作区 `firmware-v2.6.0/` 目录，两处 MD5 一致。
- 云端构建记录：Fork `deantyh/xiaozhi-esp32-1` 的 run `45751129963`（success）。

## 目标硬件
- 主控：ESP32-S3（面包板 ESP32-S3-CAM）
- 屏幕：ST7789 SPI LCD，240×320，背光 GPIO38
- 摄像头：OV2640 / OV3660（DVP，共用部分 I2S/DVP 引脚）
- 音频：I2S simplex，麦克风 WS/SCK/DIN = GPIO1/2/42，喇叭 DOUT/BCLK/LRCK = GPIO39/40/41
- 电池：GPIO3 充电检测，ADC2/CH3 电量采样（仅 `...-dl` 变体启用）

> 注意：LCD 使用 GPIO19/GPIO20，与 ESP32-S3 内置 USB D-/D+ 复用。烧录时使用板载 USB-TTL 芯片的串口，不要依赖 USB-JTAG。

## 构建方式一：项目构建脚本（推荐）
```bash
python scripts/build.py bread-compact-wifi-s3cam \
  --name bread-compact-wifi-s3cam-240x320-dl \
  --language zh-CN --wake-word nihaoxiaozhi --zip
```
脚本会设置板型/屏幕/电源选项，生成 `build/` 下的合并固件，并打包发布 ZIP。

## 构建方式二：手动 idf.py
```bash
idf.py set-target esp32s3
idf.py menuconfig
# Xiaozhi Assistant -> Board Type -> Bread Compact Wi-Fi + LCD + Camera (面包板)
# Xiaozhi Assistant -> LCD Type -> ST7789 240*320, IPS
# Xiaozhi Assistant -> Enable battery monitor and idle deep sleep -> 选中
idf.py build
# 单文件合并固件请用方式一的 build.py --zip 生成
```

## 烧录
- 单文件合并固件：地址 `0x0`，芯片 ESP32-S3，Flash 16MB，DIO，80MHz。
- 分文件镜像：
  - `0x00000 bootloader.bin`
  - `0x08000 partition-table.bin`
  - `0x0d000 ota_data_initial.bin`
  - `0x20000 xiaozhi.bin`
  - `0x800000 generated_assets.bin`

## 验证要点
1. 开机屏幕正常显示 240×320 内容，无上下/左右镜像。
2. 按键配网：连接热点后浏览器访问 `192.168.4.1`，输入 2.4GHz Wi-Fi 密码。
3. 语音对话与摄像头拍照正常；MCP `self.get_device_status` 能看到电量（`-dl` 变体）。
4. 电池供电静置 60 秒后进入低功耗表情/降背光；长时间空闲触发深度睡眠，接上充电器后退出休眠。

变更明细见 [CHANGELOG.md](CHANGELOG.md)。
