# GD32F470 工业数据采集系统

## 项目概述
基于 GD32F470VET6 的工业数据采集仪表终端，包含完整的 Bootloader 和 APP 应用。

## 系统架构

### 硬件配置
- **MCU**: GD32F470VET6（168MHz，512KB Flash，196KB SRAM）
- **系统时钟**: 168MHz
- **UART0**: PA9(TX)/PA10(RX) - 上位机通信（默认 115200bps）
- **SPI1**: PB13(SCK)/PB14(MISO)/PB15(MOSI) - Flash 通信
- **I2C0**: PB6(SCL)/PB7(SDA) - OLED 显示
- **ADC0**: 多通道采样（CH0/CH1/CH2）
- **RTC**: 时间管理
- **LED**: PA6 系统指示灯，PA7 采集工作指示灯
- **OLED**: 128x64 I2C 显示

### Flash 内存划分
| 区域 | 起始地址 | 终止地址 | 大小 |
|------|---------|---------|------|
| Bootloader | 0x08000000 | 0x0800FFFF | 64KB |
| 参数区 | 0x08010000 | 0x08010FFF | 4KB |
| APP | 0x08011000 | 0x08030FFF | 128KB |
| APP备份 | 0x08031000 | 0x08050FFF | 128KB |
| 固件暂存区 | 0x08051000 | 0x08070FFF | 128KB |

### 软件分层结构

```
Bootloader/
├── Driver/              # 硬件驱动层
│   ├── uart.c/.h
│   ├── flash.c/.h
│   ├── spi.c/.h
│   └── rcc.c/.h
├── Protocol/            # 通讯协议层
│   ├── protocol.c/.h
│   └── crc.c/.h
├── Function/            # 业务逻辑层
│   ├── bootloader.c/.h
│   ├── firmware_check.c/.h
│   └── jump_app.c/.h
├── main.c
└── startup_gd32f470.s

APP/
├── Driver/              # 硬件驱动层
│   ├── uart.c/.h
│   ├── adc.c/.h
│   ├── dac.c/.h
│   ├── flash.c/.h
│   ├── i2c.c/.h
│   ├── oled.c/.h
│   ├── led.c/.h
│   ├── rtc.c/.h
│   └── timer.c/.h
├── Protocol/            # 通讯协议层
│   ├── protocol.c/.h
│   ├── crc.c/.h
│   └── frame.c/.h
├── Function/            # 业务逻辑层
│   ├── data_collect.c/.h
│   ├── param_manage.c/.h
│   ├── alarm.c/.h
│   ├── power_manage.c/.h
│   └── business_logic.c/.h
├── main.c
└── startup_gd32f470.s
```

## 核心功能

### 1. 数据采集（3通道）
- **CH0**: 电位器采样（1-3V范围），经变比换算后上报
- **CH1**: DAC回读（PA4），经变比换算后上报
- **CH2**: 外部ADC通道，PT100温度值（换算后实际温度）

### 2. 参数持久化
- CH0/CH1 变比
- CH0/CH1 阈值
- 设备 ID
- 波特率配置
- 告警记录

### 3. 通讯协议
- 所有协议帧按十六进制结构，以 ASCII 字符串形式在串口收发
- 帧格式：帧头(2字节) + 功能码(1字节) + 数据长度(2字节) + 数据 + CRC(2字节)
- 支持单次查询、自动采集上报、参数设置等

### 4. Bootloader 功能
- 固件包校验（CRC32）
- 拒绝错误固件
- OTA在线升级
- 系统启动后5秒无升级命令时跳转APP
- 升级过程中等待10秒

### 5. 低功耗管理
- 睡眠/唤醒管理
- 定时采集功能

### 6. 显示指示
- **OLED**: 双行显示（队伍编号 + 状态）
- **LED**: 系统指示灯（1s闪烁）、采集工作指示灯（自动采集时常亮）

## 版本信息
- **固件版本**: 2.0.1.0
- **设备ID范围**: 0001-FFFE
- **默认波特率**: 115200
