# hw03 - tpu-mlir YOLOv5s Dog Detection

辰龙OS训练营 2026S 导学作业第3题：使用 tpu-mlir 跑通 dog.jpg 目标检测，产出画框结果图。

## 任务目标

使用算能 tpu-mlir 工具链，将 YOLOv5s 模型编译为 BM1684X 平台的 bmodel，并对 dog.jpg 进行目标检测，对比三种精度的检测效果。

## 运行环境

- 平台：cnb.cool CI（Docker 容器）
- 镜像：`sophgo/tpuc_dev:latest`
- 工具：`tpu_mlir` Python 包

## 三种完成方式

| 目录 | 方式 | 说明 |
|------|------|------|
| `01-ci/` | cnb.cool CI 自动执行 | push 触发，全自动产出结果 |
| `02-manual/` | 云原生开发环境手动执行 | 在 cnb.cool Workspace 中逐步操作 |
| `03-sg2002/` | 在机器人 SG2002 上执行 | 使用编译好的 bmodel 在真实硬件推理 |

## 编译流程

```
yolov5s.onnx
    │
    ├─ detect_yolov5 ──────────────────────→ dog_onnx.jpg
    │
    ├─ model_transform → yolov5s.mlir
    │       │
    │       ├─ model_deploy (F16)  → yolov5s_1684x_f16.bmodel
    │       │       └─ detect_yolov5 ──────→ dog_f16.jpg
    │       │
    │       ├─ run_calibration → yolov5s_cali_table
    │       └─ model_deploy (INT8) → yolov5s_1684x_int8_sym.bmodel
    │               └─ detect_yolov5 ──────→ dog_int8_sym.jpg
```

## 结果对比

| 文件 | 模型 | 量化类型 | 说明 |
|------|------|----------|------|
| `dog_onnx.jpg` | yolov5s.onnx | FP32 | 原始精度，基准对比 |
| `dog_f16.jpg` | yolov5s_1684x_f16.bmodel | F16 | 半精度，精度损失极小 |
| `dog_int8_sym.jpg` | yolov5s_1684x_int8_sym.bmodel | INT8 对称量化 | 最终部署格式，推理速度最快 |

## 方式1：CI 自动执行

CI配置，见 `.cnb.yml`，push 后自动触发。结果存放于 `01-ci/results/`。

**执行过程：** push 到 cnb.cool 时自动在 `sophgo/tpuc_dev:latest` 容器中依次执行：
安装 tpu_mlir → 下载模型和资源 → onnx 推理 → model_transform → model_deploy F16 → F16 推理 → run_calibration → model_deploy INT8 → INT8 推理 → commit 结果回仓库

**产出：**
- `01-ci/results/dog_onnx.jpg`
- `01-ci/results/dog_f16.jpg`
- `01-ci/results/dog_int8_sym.jpg`

## 方式2：手动执行（云原生开发环境）

在 cnb.cool Workspace 中使用 `sophgo/tpuc_dev:latest` 镜像，通过 WebIDE 终端逐步执行编译和推理命令。与方式1流程完全相同，但全程人工操作，便于观察每一步的输出和中间产物。

**产出：**
- `02-manual/results/dog_onnx.jpg` / `dog_f16.jpg` / `dog_int8_sym.jpg`
- `02-manual/models/yolov5s_1684x_f16.bmodel`
- `02-manual/models/yolov5s_1684x_int8_sym.bmodel`

## 方式3：SG2002 硬件推理

使用方式2产出的 bmodel，在 LicheeRV Nano（SG2002）上通过 sophon-sail 或 bmrt 执行推理。
结果存放于 `03-sg2002/results/`。

## 参考资料

- [tpu-mlir Quick Start](https://tpumlir.org/quick_start_en/03_onnx.html)
- [tpu-mlir GitHub](https://github.com/sophgo/tpu-mlir)
- [SG2002 产品页](https://milkv.io/chips/sg2002)
- [辰龙OS训练营 2026S](https://opencamp.cn/ChenLongOS/camp/2026S)
