# STM32 LED 闪烁实验

STM32F103C8T6 最小系统板，STM32CubeMX 生成工程，Keil MDK-ARM 编译，ST-Link V2 下载。

## 文件说明

| 文件 | 内容 |
| --- | --- |
| `main.c` | 最终版本。同时翻转 PA0、PA8、PB12~PB15、PC13，用于确认板载 LED 与面包板上外接 LED 分别由哪个引脚驱动 |
| `main_v1_PA0_PC13.c` | 第一版。外接 LED（PA0）与板载 LED（PC13）交替点亮，每 500ms 切换一次 |

两个文件均为 CubeMX 生成的完整 `main.c`，直接替换工程里 `Core/Src/main.c` 即可编译。

## 引脚与电平

| 引脚 | 接法 | 点亮条件 |
| --- | --- | --- |
| PA0 | 外接 LED 长脚（阳极）→ PA0，短脚 → 220Ω 电阻 → GND | 输出高电平 |
| PC13 | 板载 LED，已焊在板上 | 输出低电平 |

STM32F103 单个 I/O 的绝对最大输出电流为 ±25mA，裸 LED 必须串限流电阻，按 `R = (3.3V − LED 压降) / 目标电流` 取 220Ω～1kΩ。

## 编译与下载

1. Keil 打开工程，按 `F7` 编译，应为 `0 Error(s), 0 Warning(s)`；
2. 魔术棒 → `Debug` 页 → 选 `ST-Link Debugger` → `Settings` → `Port` 选 `SW`；
3. 按 `F8` 下载，Build Output 出现 `Programming Done` 与 `Verify OK` 即写入成功；
4. 按板上 `RST` 键复位。

## 演示视频

见仓库中的视频文件。

## 参考资料

1. STMicroelectronics. *RM0008 Reference Manual*. st.com.
2. STMicroelectronics. *STM32F103x8 STM32F103xB Datasheet*. st.com.
3. SparkFun Electronics. "How to Use a Breadboard." *SparkFun Learn*, learn.sparkfun.com.
