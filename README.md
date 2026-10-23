# STM32 LED 流水灯实验

基于 STM32F103C8T6 最小系统板的 GPIO 输出实验，用 STM32CubeMX 生成工程、Keil MDK-ARM 编译、ST-Link V2 下载。

## 一、实验目的

1. 掌握 STM32 GPIO 的推挽输出配置方法；
2. 理解 GPIO 输出电平与 LED 亮灭之间的关系；
3. 用延时函数控制多个 LED 按顺序点亮，实现流水灯效果。

## 二、硬件与软件环境

| 项目 | 内容 |
| --- | --- |
| 主控板 | STM32F103C8T6 最小系统板（Blue Pill） |
| 下载器 | ST-Link V2 |
| 面包板 | 830 孔无焊面包板 |
| 发光二极管 | 直插 LED |
| 限流电阻 | 220Ω ~ 1kΩ（每颗 LED 各一只） |
| 开发环境 | Keil MDK-ARM 5（ARMCC V5.06） |
| 代码生成 | STM32CubeMX（工程配置见 `led_test.ioc`） |
| 固件库 | STM32F1xx HAL Driver |

## 三、功能说明

`Core/Src/main.c` 里通过一个宏切换两种工作模式，默认使用模式 0：

```c
#define LED_MODE_4LAMP   0     /* 0 = 两灯交替流水；1 = 四灯流水 */
#define LED_STEP_MS      500   /* 每一步停留 500ms */
```

- **模式 0（默认）**：两灯交替流水。PA0 上的外接 LED 与 PC13 上的板载 LED 轮流点亮，每 500ms 切换一次，循环往复。
- **模式 1**：四灯流水。PB12、PB13、PB14、PB15 依次点亮，同一时刻只有一颗亮，走完一轮回到第一颗。

核心逻辑只有三步：先全部熄灭 → 点亮当前这一颗 → 延时 500ms，然后步进序号并循环。

```c
while (1)
{
  LED_AllOff();            /* 先全灭，保证任何时刻只有一颗灯亮 */
  LED_Step(step);          /* 点亮当前这一步对应的灯 */
  HAL_Delay(LED_STEP_MS);  /* 停留一段时间，让人眼能看到 */
  step++;
  if (step >= LED_CNT) step = 0;
}
```

## 四、接线说明

### 模式 0（本次实验采用）

| LED 引脚 | 接到 | 说明 |
| --- | --- | --- |
| 外接 LED 长脚（阳极） | PA0 所在列 | 高电平点亮 |
| 外接 LED 短脚（阴极） | 限流电阻一端 | |
| 限流电阻另一端 | GND 所在列 | 回到地，回路闭合 |
| 板载 LED | 无需接线 | 已焊在板上，由 PC13 控制，低电平点亮 |

### 模式 1（四灯流水）

四颗 LED 各自由 PB12 ~ PB15 驱动，每颗都按「GPIO 引脚 → LED 长脚 → LED 短脚 → 限流电阻 → GND」串联，四组并联在同一个 GND 上。

### 面包板取孔要点

1. 主区域每 5 个孔为一组，同组内部导通，组与组之间不导通；
2. 中间的凹槽把上下两半完全隔开，**顶边的针只能从凹槽上方取孔，底边的针只能从凹槽下方取孔**；
3. 标 `+` / `−` 的两条长轨整条导通，可作公共电源与地使用；
4. 电阻和 LED 的引脚本身是硬的，可以直接插在面包板孔里，不需要额外的杜邦线。

### 电阻取值

按 `R = (3.3V − LED 正向压降) / 目标电流` 计算。红色 LED 正向压降约 1.8 ~ 2.2V，取 5mA 时 R ≈ 260Ω；手上有 220Ω 就用 220Ω（约 5.9mA），只有 1kΩ 也能用，只是偏暗。

STM32F103 单个 I/O 的绝对最大输出电流为 ±25mA，**裸 LED 不串电阻直接接 GPIO 有烧毁引脚的风险**。

## 五、目录结构

```
.
├── Core
│   ├── Inc
│   │   ├── main.h
│   │   ├── stm32f1xx_hal_conf.h
│   │   └── stm32f1xx_it.h
│   └── Src
│       ├── main.c              ← 本实验的应用代码
│       ├── stm32f1xx_hal_msp.c
│       ├── stm32f1xx_it.c
│       └── system_stm32f1xx.c
├── Drivers                      HAL 库与 CMSIS（CubeMX 生成）
├── MDK-ARM
│   ├── led_test.uvprojx        Keil 工程文件
│   ├── led_test.uvoptx
│   └── startup_stm32f103xb.s   启动文件
├── led_test.ioc                CubeMX 工程配置
└── README.md
```

## 六、编译与下载

1. 双击 `MDK-ARM/led_test.uvprojx` 打开工程；
2. 按 `F7` 编译，Build Output 应为 `0 Error(s), 0 Warning(s)`；
3. 点魔术棒图标 → `Debug` 页 → 选择 `ST-Link Debugger` → `Settings` → `Port` 选 `SW`；
4. 按 `F8` 下载，Build Output 出现 `Programming Done` 与 `Verify OK` 表示写入成功；
5. 按板上的 `RST` 键复位，程序重新从头执行。

也可以用命令行编译：

```
UV4.exe -b MDK-ARM\led_test.uvprojx -j0 -o build.log
```

## 七、实验现象

- 烧录日志确认：`Erase Done` → `Programming Done` → `Verify OK`，即擦除、写入、回读校验三步全部通过；
- 板载 LED（PC13）以 500ms 周期亮灭，节奏稳定，周期与代码中 `LED_STEP_MS` 一致；
- 外接 LED 的接线方式与电平对应关系见第四节，实验中遇到的接线问题及其排查过程见第八节。

## 八、遇到的问题与排查记录

这一节记录实验中真实踩到的坑，结论均为实测得出。

**1. ST-Link 无法连接，Keil 报 `Target DLL has been cancelled`**

设备管理器里 `STM32 STLink`（`USB\VID_0483&PID_3748`）显示为错误状态，问题码 `CM_PROB_FAILED_INSTALL`，即驱动未能加载。解决办法是进入 `Keil\ARM\STLink\USBDriver` 目录，右键以管理员身份运行 `stlink_winusb_install.bat`，驱动装好后设备状态变为正常，Keil 即可识别。判断是否修好：Keil 的 Debug → Settings 里 Port 选 SW 后能显示出芯片 IDCODE。

**2. LED 接上后一直常亮，程序怎么写都不影响**

原因是 LED 长脚直接插在了面包板标 `+` 的常电轨上。常电轨的电位恒定，不受程序控制，所以灯永远亮。正确做法是长脚接 GPIO 引脚所在列，由程序输出的高低电平决定亮灭。

**3. LED 与电阻插在同一竖列，灯不亮**

同一竖列的 5 个孔内部是通的，长脚短脚插在同一列等于把 LED 两端短接，两端没有电位差，灯自然不亮。改法是让长脚、短脚、电阻分别落在三个不同的列上。

**4. 短脚接正极不亮，长脚接正极才亮**

这是 LED 单向导电的正常表现，不是故障。只有正向偏置（长脚电位高于短脚）时 LED 才导通发光。

**5. 面包板凹槽两侧互不导通**

板子的排针跨在凹槽两侧，顶边那排针的连接只能从凹槽上方取孔。此前从下方取孔导致「线接对了却不亮」，这是最容易踩的一处。

## 九、参考资料

1. STMicroelectronics. *RM0008 Reference Manual: STM32F101xx, STM32F102xx, STM32F103xx, STM32F105xx and STM32F107xx Advanced Arm-Based 32-Bit MCUs*. STMicroelectronics, st.com.
2. STMicroelectronics. *STM32F103x8 STM32F103xB Datasheet*. STMicroelectronics, st.com.
3. SparkFun Electronics. "How to Use a Breadboard." *SparkFun Learn*, learn.sparkfun.com/tutorials/how-to-use-a-breadboard.
4. 江协科技. "STM32入门教程-2023版 细致讲解 中文字幕." *哔哩哔哩*, www.bilibili.com/video/BV1th411z7sn.
