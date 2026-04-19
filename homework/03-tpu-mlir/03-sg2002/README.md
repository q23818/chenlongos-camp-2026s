# 方式3：SG2002 硬件推理

在 LicheeRV Nano（SG2002）上使用系统自带的 YOLOv5 模型进行目标检测。

## 环境信息

| 项目 | 值 |
|------|-----|
| 开发板 | LicheeRV Nano |
| 处理器 | SG2002 (RISC-V 1GHz) |
| 系统 | Buildroot Linux 5.10.4 |
| 模型格式 | .cvimodel (cvitek runtime) |
| 推理工具 | nn_yolov5 |

## 执行步骤

### Step 1: SSH 连接机器人

```bash
# Mac 通过 USB RNDIS 网口连接
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no root@192.168.30.17
# 密码: root
```

### Step 2: 上传测试图片

```bash
# 在 Mac 终端执行
scp -o PreferredAuthentications=password -o PubkeyAuthentication=no \
    /Users/liu/opencamp/chenlongos/2026s/homework/03-tpu-mlir/dog.jpg \
    root@192.168.30.17:/tmp/dog.jpg
```

### Step 3: 运行检测

```bash
# 在机器人终端执行
mv /tmp/dog.jpg ./
nn_yolov5 /usr/bin/yolov5s_224_int8.mud ./dog.jpg
```

### Step 4: 下载结果

```bash
# 在 Mac 终端执行
scp -o PreferredAuthentications=password -o PubkeyAuthentication=no \
    root@192.168.30.17:/root/result.jpg \
    /Users/liu/opencamp/chenlongos/2026s/homework/03-tpu-mlir/03-sg2002/results/result.jpg
```

## 检测结果

```
-- [I] result: x: 260, y: 143, w: 305, h: 281, class_id: 1, score: 0.500661,  bicycle
-- [I] result: x: 452, y: 68, w: 236, h: 96, class_id: 2, score: 0.513123,  car
-- [I] result: x: 130, y: 212, w: 188, h: 325, class_id: 16, score: 0.514514,  dog
```

成功识别出：bicycle、car、dog

## 产出

- `results/result.jpg` - 画框结果图

## 说明

SG2002 使用 cvitek 的推理框架，模型格式是 `.cvimodel`，不是 tpu-mlir 的 `.bmodel`。

系统自带了 `/usr/bin/yolov5s_224_int8.cvimodel`，可以直接使用 `nn_yolov5` 命令进行推理。

如需使用自定义 bmodel，需要先转换为 cvimodel 格式或安装 sophon-sail 库。
