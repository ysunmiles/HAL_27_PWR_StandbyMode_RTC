# HAL_27_PWR_StandbyMode_RTC

## 简介

本项目基于 STM32F103 系列 MCU 和 STM32 HAL 库，演示了低功耗待机模式（Standby Mode）的进入与唤醒机制。项目结合 RTC（实时时钟）、BKP（备份寄存器）与 0.96 寸 OLED 显示屏，实现了芯片在待机低功耗与定时/外部唤醒之间的高效切换。项目使用 STM32CubeMX 生成，并采用 CMake 进行构建。

## 主要功能

1. **RTC 实时时钟与备份域（BKP）保护**：
   * 使用外部低速晶振（LSE, 32.768 kHz）作为 RTC 时钟源。
   * 读取备份寄存器 `RTC_BKP_DR1` 的标志位（`0xA5A5`）：仅在首次上电未配置时写入初始时间（`16:20:00`），避免每次芯片复位或待机唤醒重复初始化时间，实现断电（需纽扣电池/VBAT）与待机模式下 RTC 持续走时。

2. **待机模式（Standby Mode）与双重唤醒**：
   * **RTC 闹钟唤醒**：每次系统运行均将 RTC 闹钟（Alarm A）配置为当前时间 + 5 秒，实现约 5 秒周期的定时自动唤醒。
   * **WKUP 引脚唤醒**：使能 `PA0`（`PWR_WAKEUP_PIN1`）引脚唤醒功能，支持外部上升沿（如按键）随时唤醒 MCU。
   * **调试支持**：使能 Standby 模式调试支持（`HAL_DBGMCU_EnableDBGStandbyMode()`），便于开发阶段使用调试器。
   * 进入待机模式前规范清除唤醒标志位（`__HAL_PWR_CLEAR_FLAG(PWR_FLAG_WU)`）。

3. **OLED 运行状态指示**：
   * 通过软件模拟 I2C 接口驱动 0.96 寸 OLED 显示屏。
   * 唤醒运行期间显示当前 RTC 时间及运行状态：
     * 第 1 行：`      RTC       `
     * 第 2 行：`Time: HH:MM:SS`（BCD 格式实时时间）
     * 第 4 行：`MCU Running`（运行指示，持续 500ms 后清空并进入待机）

## 运行逻辑与工作流程

1. **上电复位 / 唤醒**：芯片从复位向量重新执行 `main()`。
2. **初始化**：初始化 HAL 库、系统时钟（72MHz）、GPIO、RTC 及 OLED。
3. **读取并显示时间**：读取当前 RTC 时间并在 OLED 屏幕第 2 行刷新显示。
4. **设置下一次唤醒闹钟**：将闹钟时间设定为当前时间 + 5 秒，并开启闹钟中断。
5. **指示与延时**：OLED 第 4 行显示 `MCU Running`，延时 500ms 供观察与调试。
6. **进入待机**：清空第 4 行显示，清除唤醒标志，使能 WKUP 引脚与 Standby 调试，调用 `HAL_PWR_EnterSTANDBYMode()` 进入待机模式。
7. **唤醒后重新启动**：5 秒后 RTC 闹钟触发或按下 WKUP 按键触发唤醒，MCU 产生复位并重新从步骤 1 执行。

## 硬件连接

* **MCU**: STM32F103C8T6
* **OLED 屏幕**: 0.96 寸 OLED 显示屏 (I2C 接口，软件模拟)
  * `SDA` -> `PB8`
  * `SCL` -> `PB9`
  * `VCC` -> `3.3V`
  * `GND` -> `GND`
* **RTC 时钟源**: 外部低速晶振 (LSE, 32.768 kHz，连接至 `PC14` / `PC15`)
* **唤醒引脚 (WKUP)**: `PA0`（上升沿唤醒，可外接按键至高电平）

## 关键文件

- `CMakeLists.txt`：项目根 CMake 构建脚本。
- `config.ioc`：STM32CubeMX 项目配置文件。
- `Core/Src/main.c`：主程序入口，包含时间读取、闹钟配置、OLED 提示及待机模式控制。
- `Core/Src/rtc.c`：RTC 初始化及基于 BKP 寄存器的首次配置判断逻辑。
- `Core/Src/gpio.c`：GPIO 初始化（OLED 引脚配置）。
- `Core/Src/OLED.c`：OLED 驱动及软件 I2C 实现。
- `Core/Inc/OLED.h`：OLED 功能接口声明。
- `Drivers/`：STM32 HAL 库及 CMSIS 核心文件。

## 构建与下载

推荐使用 VS Code 的 CMake Tools 插件或通过命令行进行构建：

```bash
# 进入项目根目录
cd d:\Electronics\STM32\HAL_Projects\HAL_27_PWR_StandbyMode_RTC

# 配置构建系统 (选择 Debug 预设)
cmake --preset Debug

# 执行构建
cmake --build --preset Debug
```
