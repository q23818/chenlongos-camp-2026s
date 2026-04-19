# 方式1：CI 自动执行

## 说明

编写 `.cnb.yml`，push 到 cnb.cool 后自动触发 pipeline，在 `sophgo/tpuc_dev:latest` 容器中完整执行所有步骤，结果自动 commit 回仓库。

## 执行步骤

| 步骤 | 命令/操作 | 说明 |
|------|-----------|------|
| 1 | `pip install tpu_mlir[onnx]` | 安装 tpu-mlir 工具包 |
| 2 | `wget yolov5s.onnx` + `tpu-mlir-resource.tar` | 下载模型和资源（含 COCO2017 校准集、dog.jpg）|
| 3 | `detect_yolov5 --model yolov5s.onnx` | onnx 直接推理，得到 FP32 基准结果 |
| 4 | `model_transform` | 将 onnx 转为 tpu-mlir 中间格式 `.mlir` |
| 5 | `model_deploy --quantize F16` | 编译为 BM1684X F16 bmodel |
| 6 | `detect_yolov5 --model yolov5s_1684x_f16.bmodel` | F16 模型推理 |
| 7 | `run_calibration --input_num 100` | 用 100 张 COCO 图片生成 INT8 量化校准表 |
| 8 | `model_deploy --quantize INT8` | 编译为 BM1684X INT8 对称量化 bmodel |
| 9 | `detect_yolov5 --model yolov5s_1684x_int8_sym.bmodel` | INT8 模型推理 |
| 10 | `git commit && git push` | 将三张结果图和两个 bmodel 提交回仓库 |

## 产出

- `results/dog_onnx.jpg` — FP32 原始精度
- `results/dog_f16.jpg` — F16 半精度
- `results/dog_int8_sym.jpg` — INT8 对称量化
