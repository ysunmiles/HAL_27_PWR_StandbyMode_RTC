# HAL_24_RTC

## 简介

本项目基于 STM32F103 系列 MCU 和 STM32 HAL 库，实现了一个 RTC 实时时钟，并通过 OLED 屏幕实时显示当前的日期和时间。项目使用 STM32CubeMX 生成，并采用 CMake 进行构建。

## 主要功能

1. **RTC实时时钟**：
   * 初始化并配置 STM32 的 RTC 模块，使用外部低速晶振 (LSE, 32.768 kHz) 作为时钟源。
   * 在主循环中以 BCD 格式持续获取当前时间（时、分、秒）和日期（年、月、日）。

2. **OLED显示**：
   * 通过软件模拟 I2C 接口驱动 0.96寸 OLED 显示屏。
   * 实时显示当前时间、日期以及 RTC 的 32 位原始计数值 `CNT`。
   * 屏幕显示格式如下（共 4 行）：
     * 第 1 行：`      RTC       `
     * 第 2 行：`Time: HH:MM:SS`
     * 第 3 行：`Date: 20YY-MM-DD`
     * 第 4 行：`CNT : xxxxxxxx`
   * **OLED驱动支持的功能**：
     * 清屏 (`OLED_Clear`)
     * 显示字符 (`OLED_ShowChar`) / 字符串 (`OLED_ShowString`)
     * 显示多种格式数值 (十进制、有符号、十六进制、二进制、浮点数)
     * 显示字节数组 (`OLED_ShowArray` / `OLED_ShowHexArray`)

3. **RTC计数器读取**：
   * 调用 `RTC_ReadTimeCounter()` 直接读取 RTC 的 32 位秒计数器值，并以十六进制实时显示在 OLED 上。

## 硬件连接

* **MCU**: STM32F103C8T6
* **OLED**: 0.96寸 OLED 显示屏 (I2C 接口，软件模拟)
  * `SCL` -> `PB8`
  * `SDA` -> `PB9`
  * `VCC` -> `3.3V`
  * `GND` -> `GND`
* **RTC时钟源**: 外部低速晶振 (LSE, 32.768 kHz，连接至 PC14 / PC15)

## 使用说明

1. 将代码下载到 STM32F103C8T6 开发板。
2. 按照引脚定义正确连接 OLED 显示屏。
3. 上电后，OLED 屏幕将显示实时时间、日期与计数值。

> **注意**: 
> - 当前工程默认初始时间设为 `2026-08-16 10:12:00`。
> - 由于未加入备份域（BKP）判断逻辑，每次芯片复位或断电重启均会重新加载初始时间。如需自定义初始时间，请在 `Core/Src/rtc.c` 中的 `MX_RTC_Init()` 函数或 STM32CubeMX 中修改。

## 关键文件

- `CMakeLists.txt`：项目根 CMake 构建脚本。
- `config.ioc`：STM32CubeMX 项目配置文件。
- `Core/Src/main.c`：主程序入口，包含主循环读取与显示逻辑。
- `Core/Src/rtc.c`：RTC 初始化及配置。
- `Core/Src/OLED.c`：OLED 驱动及软件 I2C 实现。
- `Core/Inc/OLED.h`：OLED 功能接口声明。
- `Drivers/`：STM32 HAL 库及 CMSIS 核心文件。

## 构建与下载

推荐使用 VS Code 的 CMake Tools 插件或通过命令行进行构建：

```bash
# 进入项目根目录
cd d:\Electronics\STM32\HAL_Projects\HAL_24_RTC

# 配置构建系统 (选择 Debug 预设)
cmake --preset Debug

# 执行构建
cmake --build --preset Debug
