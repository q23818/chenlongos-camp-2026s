# SPI 协议深入学习文档

> 结合辰龙OS训练营（2026S）实战场景：算能 SG2002 芯片 + ArceOS + ESP32-CAM SPI 传输

---

## 1. SPI 协议基础

### 1.1 什么是 SPI

SPI（Serial Peripheral Interface，串行外设接口）是一种同步串行通信协议，由 Motorola 在 1980 年代提出。广泛用于微控制器与外设之间的短距离高速通信。

**参考资料：**
- [Analog Devices: Introduction to SPI Interface](https://www.analog.com/en/analog-dialogue/articles/introduction-to-spi-interface.html)
- [Wikipedia: Serial Peripheral Interface](https://en.wikipedia.org/wiki/Serial_Peripheral_Interface)

### 1.2 四根信号线

| 信号线 | 全称 | 方向 | 说明 |
|--------|------|------|------|
| SCLK | Serial Clock | 主→从 | 时钟信号，由主设备产生 |
| MOSI | Master Out Slave In | 主→从 | 主设备发送数据 |
| MISO | Master In Slave Out | 从→主 | 从设备发送数据 |
| CS/SS | Chip Select / Slave Select | 主→从 | 低电平有效，选中从设备 |

**全双工**：MOSI 和 MISO 同时工作，主从设备可以同时收发数据。

### 1.3 主从架构

```
         ┌─────────────┐
         │  主设备      │
         │  (SG2002)   │
         └──┬──┬──┬──┬─┘
            │  │  │  │
          SCLK MOSI MISO CS
            │  │  │  │
         ┌──┴──┴──┴──┴─┐
         │  从设备      │
         │  (ESP32-CAM)│
         └─────────────┘
```

多个从设备时，每个从设备有独立的 CS 线，SCLK/MOSI/MISO 共享。

---

## 2. 工作模式：CPOL 与 CPHA

SPI 有四种工作模式，由两个参数决定：

| 参数 | 含义 | 值 |
|------|------|----|
| CPOL | Clock Polarity，时钟极性 | 0 = 空闲低电平；1 = 空闲高电平 |
| CPHA | Clock Phase，时钟相位 | 0 = 第一个边沿采样；1 = 第二个边沿采样 |

| 模式 | CPOL | CPHA | 说明 |
|------|------|------|------|
| Mode 0 | 0 | 0 | 最常用，空闲低，上升沿采样 |
| Mode 1 | 0 | 1 | 空闲低，下降沿采样 |
| Mode 2 | 1 | 0 | 空闲高，下降沿采样 |
| Mode 3 | 1 | 1 | 空闲高，上升沿采样 |

**关键原则**：主从设备必须使用相同的模式，否则数据错乱。

**时序图（Mode 0）：**
```
CS   ‾‾‾‾|_________________________|‾‾‾‾
SCLK ______|‾|_|‾|_|‾|_|‾|_|‾|_|______
MOSI ______| D7 | D6 | D5 | D4 |...___
MISO ______| D7 | D6 | D5 | D4 |...___
```

**参考资料：**
- [SPI Mode 详解 - SparkFun](https://learn.sparkfun.com/tutorials/serial-peripheral-interface-spi/all)

---

## 3. SPI 与其他协议对比

| 特性 | SPI | I2C | UART |
|------|-----|-----|------|
| 信号线数 | 4（+每从设备1根CS） | 2 | 2 |
| 速率 | 高（可达数百MHz） | 中（最高5MHz） | 低（通常<10Mbps） |
| 全双工 | ✅ | ❌ | ✅ |
| 多从设备 | 需多根CS线 | 地址寻址 | 不支持 |
| 距离 | 短（板内） | 短（板内） | 较长 |
| 适用场景 | 高速外设（Flash、摄像头、显示屏） | 传感器、低速外设 | 调试、模块通信 |

**结论**：ESP32-CAM 传输图像数据选用 SPI，正是因为其高速全双工特性。

---

## 4. 算能 SG2002 的 SPI 控制器

### 4.1 芯片概览

SG2002 是算能（Sophgo）推出的边缘 AI 芯片：
- 主核：RISC-V C906（1GHz）+ ARM Cortex-A53
- 副核：RISC-V C906（700MHz）
- MCU：8051
- NPU：1 TOPS
- RAM：256MB（片内集成）

**参考资料：**
- [SG2002 产品页 - Milk-V](https://milkv.io/chips/sg2002)
- [SG2002 TRM（技术参考手册）- sophgo/sophgo-doc](https://github.com/sophgo/sophgo-doc/releases/tag/sg2002-trm-v1.02)

### 4.2 SPI 控制器特性

SG2002 集成多个 SPI 控制器，主要特性：
- 支持 SPI Master / Slave 模式
- 支持 Motorola SPI、TI SSP、NS Microwire 三种帧格式
- 数据位宽：4~16 bit 可配置
- 支持 DMA 传输（减少 CPU 占用）
- 支持 FIFO 缓冲

### 4.3 关键寄存器

| 寄存器 | 偏移 | 说明 |
|--------|------|------|
| CTRLR0 | 0x00 | 控制寄存器0：帧格式、数据位宽、CPOL/CPHA |
| CTRLR1 | 0x04 | 控制寄存器1：接收数据帧数 |
| SSIENR | 0x08 | SPI 使能 |
| SER | 0x10 | 从设备选择（CS） |
| BAUDR | 0x14 | 波特率分频 |
| TXFLTR | 0x18 | 发送 FIFO 阈值 |
| RXFLTR | 0x1C | 接收 FIFO 阈值 |
| SR | 0x28 | 状态寄存器（忙/FIFO状态） |
| DR | 0x60 | 数据寄存器（读写数据） |

**操作流程：**
1. 禁用 SPI（SSIENR = 0）
2. 配置 CTRLR0（模式、位宽）
3. 配置 BAUDR（时钟频率）
4. 使能 SPI（SSIENR = 1）
5. 拉低 CS（SER 对应位置1）
6. 写 DR 发送 / 读 DR 接收
7. 等待 SR.BUSY = 0
8. 拉高 CS

---

## 5. ArceOS 中的 SPI 驱动开发

### 5.1 ArceOS 驱动框架

ArceOS 采用组件化架构，驱动以 crate 形式存在：

```
arceos/
├── modules/
│   └── axhal/          # 硬件抽象层
├── crates/
│   └── driver_*/       # 各类驱动 crate
└── platforms/          # 平台相关配置
```

目前 ArceOS 尚无 SG2002 SPI 驱动，训练营第二周的核心任务就是实现它。

**参考资料：**
- [ArceOS 仓库](https://github.com/arceos-org/arceos)
- [ArceOS 设计文档](https://arceos.org)

### 5.2 Rust 驱动实现要点

```rust
// SPI 控制器基地址（SG2002 示例）
const SPI0_BASE: usize = 0x04180000;

// 寄存器偏移
const CTRLR0: usize = 0x00;
const SSIENR: usize = 0x08;
const DR:     usize = 0x60;

// 读写寄存器的基本操作
unsafe fn write_reg(base: usize, offset: usize, val: u32) {
    let ptr = (base + offset) as *mut u32;
    ptr.write_volatile(val);
}

unsafe fn read_reg(base: usize, offset: usize) -> u32 {
    let ptr = (base + offset) as *const u32;
    ptr.read_volatile()
}
```

**关键点：**
- 必须使用 `read_volatile` / `write_volatile`，防止编译器优化掉寄存器访问
- 操作前后需要内存屏障（`core::sync::atomic::fence`）
- DMA 传输需要物理地址连续的缓冲区

### 5.3 embedded-hal 抽象

Rust 嵌入式生态有标准的 SPI trait：

```rust
// embedded-hal v1.0
use embedded_hal::spi::SpiBus;

impl SpiBus for Sg2002Spi {
    fn transfer(&mut self, read: &mut [u8], write: &[u8]) -> Result<(), Self::Error> {
        // 实现全双工传输
    }
}
```

**参考资料：**
- [embedded-hal SPI trait](https://docs.rs/embedded-hal/latest/embedded_hal/spi/index.html)

---

## 6. ESP32-CAM 通过 SPI 传输图像

### 6.1 场景描述

训练营第四周实战：
- **ESP32-CAM**：作为 SPI 从设备，采集摄像头图像后通过 SPI 发送
- **SG2002（辰龙OS）**：作为 SPI 主设备，接收图像数据后送入 TPU 推理

```
OV2640摄像头 → ESP32-CAM → SPI总线 → SG2002 → TPU(YOLOv8推理) → 结果输出
```

### 6.2 ESP32-CAM SPI 从设备配置

ESP32 支持 SPI Slave 模式，典型配置：

```c
// ESP32 端（Arduino/ESP-IDF）
spi_slave_interface_config_t slvcfg = {
    .mode = 0,              // SPI Mode 0
    .spics_io_num = CS_PIN,
    .queue_size = 3,
    .flags = 0,
};
spi_slave_initialize(VSPI_HOST, &buscfg, &slvcfg, DMA_CHAN);
```

### 6.3 图像传输协议设计

由于 SPI 无内置帧同步机制，需要自定义协议：

```
┌──────────┬──────────┬──────────┬──────────────────┬──────────┐
│ 帧头     │ 宽度     │ 高度     │ 图像数据          │ 校验和   │
│ 0xAA55   │ 2 bytes  │ 2 bytes  │ width*height*2   │ 2 bytes  │
└──────────┴──────────┴──────────┴──────────────────┴──────────┘
```

### 6.4 DMA 优化

图像数据量大（如 320×240 RGB565 = 150KB），必须用 DMA：
- CPU 发起 DMA 传输请求
- DMA 控制器直接将 SPI FIFO 数据写入内存
- 传输完成后触发中断通知 CPU
- CPU 零拷贝将数据传给 TPU

**参考资料：**
- [ESP32 SPI Slave 文档](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/spi_slave.html)

---

## 7. SPI 与 AI 推理 Pipeline

### 7.1 完整数据流

```
ESP32-CAM
  │ SPI（~40MHz）
  ▼
SG2002 SPI控制器
  │ DMA（零拷贝）
  ▼
DDR 内存缓冲区
  │ 预处理（resize/normalize）
  ▼
TPU（1 TOPS）
  │ YOLOv8推理
  ▼
检测结果（bounding box）
```

### 7.2 性能关键点

| 环节 | 瓶颈 | 优化手段 |
|------|------|----------|
| SPI 传输 | 带宽（40MHz × 4bit = 20MB/s） | 提高时钟频率、使用 Quad-SPI |
| 内存拷贝 | CPU 占用 | DMA 传输 |
| 预处理 | CPU 计算 | SIMD 指令 / 硬件加速 |
| TPU 推理 | 模型大小 | INT8 量化 |

### 7.3 与 tpu-mlir 的关联

题目3（tpu-mlir）是在 Linux 下验证模型推理流程，而训练营最终目标是把这个推理流程移植到辰龙OS上，输入数据来源正是 SPI 传输的摄像头图像。两道题是同一 pipeline 的不同环节。

**参考资料：**
- [tpu-mlir 仓库](https://github.com/sophgo/tpu-mlir)

---

## 8. 总结

| 知识点 | 在训练营中的应用 |
|--------|-----------------|
| SPI 四线协议 | 理解 ESP32-CAM ↔ SG2002 的物理连接 |
| CPOL/CPHA 模式 | 配置主从设备使用相同模式，避免数据错误 |
| SG2002 SPI 寄存器 | 编写裸机驱动，直接操作硬件 |
| DMA 传输 | 高效传输图像数据，降低 CPU 占用 |
| embedded-hal trait | 编写可复用的 Rust SPI 驱动 |
| 完整 Pipeline | SPI图像采集 → TPU推理 → 结果输出 |

---

## 参考链接汇总

1. [SPI 协议入门 - Analog Devices](https://www.analog.com/en/analog-dialogue/articles/introduction-to-spi-interface.html)
2. [SPI 详解 - SparkFun](https://learn.sparkfun.com/tutorials/serial-peripheral-interface-spi/all)
3. [Wikipedia: SPI](https://en.wikipedia.org/wiki/Serial_Peripheral_Interface)
4. [SG2002 产品页 - Milk-V](https://milkv.io/chips/sg2002)
5. [SG2002 TRM v1.02 - sophgo/sophgo-doc](https://github.com/sophgo/sophgo-doc/releases/tag/sg2002-trm-v1.02)
6. [ArceOS 仓库](https://github.com/arceos-org/arceos)
7. [embedded-hal SPI trait](https://docs.rs/embedded-hal/latest/embedded_hal/spi/index.html)
8. [ESP32 SPI Slave 文档](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/spi_slave.html)
9. [tpu-mlir 仓库](https://github.com/sophgo/tpu-mlir)
10. [辰龙OS训练营 2026S](https://opencamp.cn/ChenLongOS/camp/2026S)
11. [夏豪老师的spi_protocol_study.md](https://github.com/saymain/DORA_IOS/blob/main/spi_protocol_study.md)
12. [关于SPI协议，看这一篇文章就够了！](https://mp.weixin.qq.com/s/MfIgX2xU2eSU30NKO_vMYw?scene=1)
