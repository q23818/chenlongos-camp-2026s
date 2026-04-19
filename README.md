# chenlongos-camp-2026s

辰龙OS训练营 2026春季 学习笔记与作业仓库

## 课程信息

- **课程**：[辰龙OS训练营 2026S](https://opencamp.cn/ChenLongOS/camp/2026S)
- **时间**：2026/04/19 - 2026/05/17
- **主办方**：辰龙操作系统开源社区

## 仓库结构

```
├── README.md           # 本文件
├── do.md               # 学习笔记（详细操作记录）
├── .cnb.yml            # cnb.cool CI 配置
└── homework/           # 作业目录
    ├── 01-arceos-helloworld/   # 作业1：ArceOS helloworld
    ├── 02-spi-protocol/        # 作业2：SPI 协议学习文档
    └── 03-tpu-mlir/            # 作业3：tpu-mlir YOLOv5s 检测
```

## 作业完成情况

| 作业 | 内容 | 状态 |
|------|------|------|
| hw01 | ArceOS helloworld on QEMU RISC-V | ✅ 完成 |
| hw02 | SPI 协议深入学习文档 | ✅ 完成 |
| hw03 | tpu-mlir YOLOv5s dog.jpg 检测 | ✅ 完成（三种方式） |

## 作业链接

- [作业1：ArceOS helloworld](./homework/01-arceos-helloworld/)
- [作业2：SPI 协议学习文档](./homework/02-spi-protocol/spi-learning.md)
- [作业3：tpu-mlir YOLOv5s 检测](./homework/03-tpu-mlir/)

## CI/CD

本仓库使用 [cnb.cool](https://cnb.cool) 进行 CI 构建，配置见 [`.cnb.yml`](./.cnb.yml)。

镜像同步：GitHub ↔ cnb.cool
