# hw03 - tpu-mlir YOLOv5s Dog Detection

辰龙OS训练营 2026S 导学作业第3题：使用 tpu-mlir 跑通 dog.jpg 目标检测，产出三张画框结果图。

## 任务目标

使用算能 tpu-mlir 工具链，将 YOLOv5s 模型编译为 BM1684X 平台的 bmodel，并对 dog.jpg 进行目标检测，对比三种精度的检测效果。

## 运行环境

- 平台：cnb.cool CI（Docker 容器）
- 镜像：`sophgo/tpuc_dev:latest`
- 工具：`tpu_mlir` Python 包

## 完整流程

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

## 结果图片

| 文件 | 模型 | 量化类型 | 说明 |
|------|------|----------|------|
| `dog_onnx.jpg` | yolov5s.onnx | FP32 | 原始精度，基准对比 |
| `dog_f16.jpg` | yolov5s_1684x_f16.bmodel | F16 | 半精度，精度损失极小 |
| `dog_int8_sym.jpg` | yolov5s_1684x_int8_sym.bmodel | INT8 对称量化 | 最终部署格式，推理速度最快 |

结果图片由 CI 产出，存放于 `results/` 目录。

## CI 配置

见 `.cnb.yml`，push 后自动触发。

## 参考资料

- [tpu-mlir Quick Start - Compile ONNX Model](https://tpumlir.org/quick_start_en/03_onnx.html)
- [tpu-mlir GitHub](https://github.com/sophgo/tpu-mlir)
- [SG2002 产品页](https://milkv.io/chips/sg2002)
- [辰龙OS训练营 2026S](https://opencamp.cn/ChenLongOS/camp/2026S)
