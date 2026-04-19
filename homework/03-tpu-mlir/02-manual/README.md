# 方式2：云原生开发环境手动执行

## 说明

在 cnb.cool Workspace（`sophgo/tpuc_dev:latest` 容器）中，通过 WebIDE 终端逐步手动执行每一条命令。与方式1流程完全相同，但全程人工操作，便于观察每一步的输出和中间产物。

## 执行步骤

```bash
# Step 1：安装 tpu_mlir（安装 tpu-mlir 工具包及 onnx 依赖）
pip install tpu_mlir[onnx]

# Step 2：准备工作目录（下载模型和资源）
cd /workspace
wget -q https://github.com/ultralytics/yolov5/releases/download/v6.0/yolov5s.onnx
wget -q https://github.com/sophgo/tpu-mlir/releases/latest/download/tpu-mlir-resource.tar
tar -xf tpu-mlir-resource.tar && mv regression/ tpu-mlir-resource/
mkdir -p model_yolov5s/workspace
cp yolov5s.onnx model_yolov5s/
cp -rf tpu-mlir-resource/dataset/COCO2017 model_yolov5s/
cp -rf tpu-mlir-resource/image model_yolov5s/

# Step 3：onnx 推理（FP32 基准，直接用原始 onnx 模型）
cd /workspace/model_yolov5s
detect_yolov5 --input ./image/dog.jpg --model ./yolov5s.onnx --output dog_onnx.jpg

# Step 4：onnx → mlir（将 onnx 转为 tpu-mlir 中间格式）
cd /workspace/model_yolov5s/workspace
model_transform \
  --model_name yolov5s --model_def ../yolov5s.onnx \
  --input_shapes [[1,3,640,640]] \
  --mean 0.0,0.0,0.0 --scale 0.0039216,0.0039216,0.0039216 \
  --keep_aspect_ratio --pixel_format rgb \
  --output_names 350,498,646 \
  --test_input ../image/dog.jpg --test_result yolov5s_top_outputs.npz \
  --mlir yolov5s.mlir

# Step 5：编译 F16 bmodel（半精度，适合 BM1684X）
model_deploy \
  --mlir yolov5s.mlir --quantize F16 --processor bm1684x \
  --test_input yolov5s_in_f32.npz --test_reference yolov5s_top_outputs.npz \
  --tolerance 0.99,0.99 --model yolov5s_1684x_f16.bmodel

# Step 6：F16 推理
cd /workspace/model_yolov5s
detect_yolov5 --input ./image/dog.jpg --model ./workspace/yolov5s_1684x_f16.bmodel --output dog_f16.jpg

# Step 7：INT8 校准（用 100 张 COCO 图片统计量化参数）
cd /workspace/model_yolov5s/workspace
run_calibration yolov5s.mlir --dataset ../COCO2017 --input_num 100 -o yolov5s_cali_table

# Step 8：编译 INT8 bmodel（INT8 对称量化，推理速度最快）
model_deploy \
  --mlir yolov5s.mlir --quantize INT8 \
  --calibration_table yolov5s_cali_table --processor bm1684x \
  --test_input yolov5s_in_f32.npz --test_reference yolov5s_top_outputs.npz \
  --tolerance 0.85,0.45 --model yolov5s_1684x_int8_sym.bmodel

# Step 9：INT8 推理
cd /workspace/model_yolov5s
detect_yolov5 --input ./image/dog.jpg --model ./workspace/yolov5s_1684x_int8_sym.bmodel --output dog_int8_sym.jpg

# Step 10：提交结果到仓库
cd /workspace
mkdir -p homework/03-tpu-mlir/02-manual/results homework/03-tpu-mlir/02-manual/models
cp model_yolov5s/dog_*.jpg homework/03-tpu-mlir/02-manual/results/
cp model_yolov5s/workspace/*.bmodel homework/03-tpu-mlir/02-manual/models/
git add homework/03-tpu-mlir/02-manual/
git commit -m "hw03: add manual execution results and bmodels"
git push
```

## 产出

- `results/dog_onnx.jpg` — FP32 原始精度
- `results/dog_f16.jpg` — F16 半精度
- `results/dog_int8_sym.jpg` — INT8 对称量化
- `models/yolov5s_1684x_f16.bmodel` — 供方式3（SG2002）使用
- `models/yolov5s_1684x_int8_sym.bmodel` — 供方式3（SG2002）使用
