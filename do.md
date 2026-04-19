
# Learning Notes for ChenLongOS Camp 2026 Spring

2026-04-18 11:53:48

## 0. about the course

https://opencamp.cn/ChenLongOS/camp/2026S

**2026 春季辰龙操作系统训练营**

**时间**  ： 2026/04/19-2026/05/17

**主办方**： 辰龙操作系统开源社区

-------------------------------------------------------
## 1. init repo

**创建仓库**

create repo: https://github.com/q23818/chenlongos-camp-2026s

```bash
cd /Users/liu/opencamp/chenlongos/2026s/
echo "# chenlongos-camp-2026s" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:q23818/chenlongos-camp-2026s.git
git push -u origin main

echo "*.swp\n*.un~\n*~" > .gitignore
git add .gitignore
git commit -m "add gitignore"
git push

```

-------------------------------------------------------
## 2. guidance learning

**课程导学**

### homework tasks

1. 按照下面这个仓库的 readme 文档在 qemu riscv 下跑起来 arceos helloworld。

    考核：截图启动命令和 helloworld 输出。 https://github.com/arceos-org/arceos

2. （选做）深入了解 spi 协议，

    考核：产出一份详细学习的文档。

3. （选做）学习 tpu-mlir 的基本使用方法（如果对神经网络不熟悉的同学可以先了解一下神经网络基础知识）

    考核：跑通 readme 中 dog.jpg 的识别，得到一个画框结果图片

    参考链接：https://github.com/sophgo/tpu-mlir

4. 导学作业提交：

    作业提交到这个文档： https://docs.qq.com/sheet/DUEVHd3hQUkxoWGR6?tab=BB08J2


-------------------------------------------------------
### task1 solution

#### 环境准备

**Step 0：检查环境**

> Purpose: 检查 Rust RISC-V target 和 QEMU 环境

```bash
rustup target list --installed | grep riscv; echo "---"; qemu-system-riscv64 --version 2>/dev/null || echo "qemu not found"; echo "---"; rustc --version; cargo --version
```
> 环境缺两样东西，需要先装好再跑 ArceOS。
缺少的环境：
1. QEMU（qemu-system-riscv64）
2. Rust RISC-V target（riscv64gc-unknown-none-elf）

**Step 1：安装工具链**

```bash
cargo install cargo-binutils axconfig-gen cargo-axplat
brew install qemu
rustup target add riscv64gc-unknown-none-elf
```

验证：
```bash
qemu-system-riscv64 --version   # QEMU emulator version 10.2.2
rustup target list --installed  # 应包含 riscv64gc-unknown-none-elf
```

**Step 2：克隆 ArceOS 并运行**

```bash
cd /Users/liu/opencamp/chenlongos/2026s/
git clone https://github.com/arceos-org/arceos.git
cd arceos
make A=examples/helloworld ARCH=riscv64 run
```

**Step 3：截图存入仓库**

```bash
mkdir -p /Users/liu/opencamp/chenlongos/2026s/homework/01-arceos-helloworld/
# 截图放入该目录
```

#### 结果

✅ 2026-04-18 成功在 QEMU RISC-V 下运行 ArceOS helloworld。

截图存放于 `homework/01-arceos-helloworld/`：
- screenshot1.png ~ screenshot5.png



-------------------------------------------------------
## 3. homework task2 - SPI 协议深入学习

**考核**：产出一份详细学习文档

文档路径：`homework/02-spi-protocol/spi-learning.md`

**文档涵盖：**
1. SPI 协议基础（四线、主从架构）
2. 四种工作模式（CPOL/CPHA）
3. 与 I2C/UART 对比
4. 算能 SG2002 SPI 控制器（寄存器、操作流程）
5. ArceOS 驱动开发要点（Rust 实现、embedded-hal）
6. ESP32-CAM SPI 图像传输实战
7. SPI → TPU 推理完整 Pipeline




-------------------------------------------------------
## 4. homework task3 - tpu-mlir YOLOv5s Dog Detection

**考核**：跑通 dog.jpg 识别，得到画框结果图片

参考：https://github.com/sophgo/tpu-mlir

### 三种完成方式

**方式1：cnb.cool CI 自动执行**
- 配置 `.cnb.yml`，push 触发全自动流程
- 结果：`homework/03-tpu-mlir/01-ci/results/`
- 状态：✅ 已完成（dog_onnx.jpg / dog_f16.jpg / dog_int8_sym.jpg）

**方式2：云原生开发环境手动执行**
- 在 cnb.cool Workspace（sophgo/tpuc_dev 容器）中逐步操作
- 结果：`homework/03-tpu-mlir/02-manual/results/`
- bmodel：`homework/03-tpu-mlir/02-manual/models/`
- 状态：⬜ 待完成

**方式3：SG2002 硬件推理**
- 使用方式2产出的 bmodel，在 LicheeRV Nano（SG2002）上推理
- 结果：`homework/03-tpu-mlir/03-sg2002/results/`
- 状态：⬜ 待完成

### 编译流程

```
yolov5s.onnx → model_transform → yolov5s.mlir
    ├─ model_deploy F16  → yolov5s_1684x_f16.bmodel
    └─ run_calibration + model_deploy INT8 → yolov5s_1684x_int8_sym.bmodel
```
