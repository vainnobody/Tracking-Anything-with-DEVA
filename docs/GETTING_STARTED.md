# DEVA 部署与使用指南 / Getting Started with DEVA

## 1. 项目简介 / Overview

中文：DEVA 是一个将图像级分割模型与时序传播模块解耦的视频分割框架，适合用于开放词汇视频分割、文本提示跟踪，以及基于 SAM 的自动视频分割。这个仓库已经提供了模型推理、Gradio 演示、训练和评测入口，适合快速试跑示例视频或接入你自己的视频帧目录。

English: DEVA is a decoupled video segmentation framework that combines image-level segmentation models with a temporal propagation module. It is suitable for open-vocabulary video segmentation, text-prompted tracking, and SAM-based automatic video segmentation. This repository already includes inference scripts, a Gradio demo, training, and evaluation entry points, so you can quickly test the bundled example or run it on your own frame folders.

## 2. 环境要求 / Requirements

中文：项目 README 中说明该仓库主要在 Ubuntu 上验证。建议准备带 CUDA 的 Linux 环境，并提前安装与你显卡和 CUDA 版本匹配的 PyTorch 与 torchvision。基础要求如下：

- Python 3.9+
- PyTorch 1.12+ 和对应版本的 `torchvision`
- 建议使用 NVIDIA GPU 和可用的 CUDA 环境

English: The main README states that the project is tested on Ubuntu. A Linux environment with CUDA is recommended, and you should install a PyTorch and torchvision build that matches your GPU and CUDA version in advance. The baseline requirements are:

- Python 3.9+
- PyTorch 1.12+ with matching `torchvision`
- An NVIDIA GPU and working CUDA setup are strongly recommended

## 3. 部署步骤 / Deployment

### 3.1 克隆仓库 / Clone the repository

```bash
git clone https://github.com/hkchengrex/Tracking-Anything-with-DEVA.git
cd Tracking-Anything-with-DEVA
```

### 3.2 安装 Python 依赖 / Install Python dependencies

中文：在已经安装好 PyTorch 的前提下，执行下面命令安装仓库依赖。

English: After installing PyTorch, run the command below to install the repository dependencies.

```bash
pip install -e .
```

中文：如果遇到 `File "setup.py" not found`，先升级 `pip` 后再重试。

English: If you hit `File "setup.py" not found`, upgrade `pip` and try again.

```bash
pip install --upgrade pip
```

### 3.3 下载预训练模型 / Download pretrained checkpoints

中文：仓库已经提供模型下载脚本，会把 DEVA、GroundingDINO、SAM、HQ-SAM 和 MobileSAM 相关权重下载到 `./saves/`。

English: The repository already includes a download script that fetches the DEVA, GroundingDINO, SAM, HQ-SAM, and MobileSAM checkpoints into `./saves/`.

```bash
bash scripts/download_models.sh
```

### 3.4 文本提示模式额外依赖 / Extra dependency for text-prompted mode

中文：文本提示模式依赖作者维护的 Grounded-Segment-Anything 分支，请根据其仓库说明完成安装：

[https://github.com/hkchengrex/Grounded-Segment-Anything](https://github.com/hkchengrex/Grounded-Segment-Anything)

安装完成后，建议执行下面的检查命令，确认 Grounding DINO 没有静默安装失败：

```bash
python -c "from groundingdino.util.inference import Model as GroundingDINOModel"
```

如果提示只能在 CPU 模式运行，请确认安装 Grounding DINO 时已经正确设置 `CUDA_HOME`。

English: The text-prompted mode depends on the authors' Grounded-Segment-Anything fork. Follow the installation steps in:

[https://github.com/hkchengrex/Grounded-Segment-Anything](https://github.com/hkchengrex/Grounded-Segment-Anything)

After installation, run the following check to make sure Grounding DINO did not fail silently:

```bash
python -c "from groundingdino.util.inference import Model as GroundingDINOModel"
```

If it falls back to CPU-only execution, make sure `CUDA_HOME` was set correctly while installing Grounding DINO.

### 3.5 可选优化依赖 / Optional optimization dependency

中文：在半在线模式中，Gurobi 可用于更快地求解整数规划问题。没有 license 时，代码会退回到 `PuLP`，但速度更慢，且作者没有像 Gurobi 那样充分测试。

English: In semi-online mode, Gurobi can be used to solve the integer program more efficiently. If no license is available, the code falls back to `PuLP`, which is slower and less battle-tested by the authors.

## 4. 使用方法 / Usage

### 4.1 启动 Gradio 演示 / Launch the Gradio demo

中文：如果你想先通过网页界面试用，可以直接启动 Gradio。程序启动后，终端会输出一个本地访问地址。

English: If you want to try the project through a browser UI first, start the Gradio demo. The terminal will print a local URL after startup.

```bash
python demo/demo_gradio.py
```

中文：如果在远程服务器上运行，可以结合 SSH 端口转发访问页面。

English: If you are running on a remote server, use SSH port forwarding to access the UI from your local machine.

### 4.2 文本提示模式 / Text-prompted mode

中文：文本提示模式会先用 GroundingDINO 根据文本生成框，再由 SAM 生成掩码，并与 DEVA 的传播结果融合。仓库自带示例帧目录 `./example/vipseg/images/12_1mWNahzcsAc`，可以直接试跑：

English: In text-prompted mode, GroundingDINO first predicts boxes from the text prompt, SAM turns them into masks, and DEVA merges them with temporal propagation. The repository already includes `./example/vipseg/images/12_1mWNahzcsAc`, which you can use directly:

```bash
python demo/demo_with_text.py --chunk_size 4 \
--img_path ./example/vipseg/images/12_1mWNahzcsAc \
--amp --temporal_setting semionline \
--size 480 \
--output ./example/output \
--prompt person.hat.horse
```

中文：`--prompt` 里的类别用英文句点分隔，例如 `person.hat.horse`。

English: Separate class names in `--prompt` with periods, for example `person.hat.horse`.

### 4.3 自动分割模式 / Automatic mode

中文：自动模式不依赖文本提示，而是在未分割区域上生成网格点，调用 SAM 自动生成候选分割结果：

English: Automatic mode does not rely on a text prompt. Instead, it samples a grid of points in unsegmented regions and uses SAM to generate candidate masks:

```bash
python demo/demo_automatic.py --chunk_size 4 \
--img_path ./example/vipseg/images/12_1mWNahzcsAc \
--amp --temporal_setting semionline \
--size 480 \
--output ./example/output
```

## 5. 常用参数说明 / Common arguments

中文：下面这些参数是命令行使用时最常调整的选项。

English: These are the parameters you are most likely to tune in the command-line scripts.

- `--img_path`: 输入帧目录 / Input frame directory
- `--output`: 输出目录；推理结果会写入这里 / Output directory where inference results are written
- `--amp`: 启用混合精度，通常更快且更省显存 / Enables mixed precision for better speed and lower memory usage on most modern GPUs
- `--chunk_size`: 并行处理的对象数量；更大通常更快，但更占显存 / Number of objects processed in parallel; larger is often faster but uses more memory
- `--size`: 传播模块内部处理分辨率，默认 `480` / Internal processing resolution for the propagation module, default `480`
- `--temporal_setting`: 时序模式，可选 `semionline` 或 `online` / Temporal mode, either `semionline` or `online`
- `--detection_every`: 每隔多少帧执行一次检测 / Number of frames between detections
- `--max_missed_detection_count`: 连续多少次未检测到后删除对象 / Number of consecutive missed detections before an object is removed

中文：文本提示模式额外常用 `--prompt`；自动模式常用 `SAM_NUM_POINTS_PER_SIDE`、`SAM_NUM_POINTS_PER_BATCH`、`SAM_PRED_IOU_THRESHOLD` 和 `--suppress_small_objects`。更完整的参数定义可参考 `deva/inference/eval_args.py` 和 `deva/ext/ext_eval_args.py`。

English: Text-prompted mode additionally relies on `--prompt`. Automatic mode commonly uses `SAM_NUM_POINTS_PER_SIDE`, `SAM_NUM_POINTS_PER_BATCH`, `SAM_PRED_IOU_THRESHOLD`, and `--suppress_small_objects`. For the full parameter list, refer to `deva/inference/eval_args.py` and `deva/ext/ext_eval_args.py`.

## 6. 输出结果说明 / Output layout

中文：以 `--output ./example/output` 为例，命令行 demo 运行完成后通常会生成以下内容：

English: Using `--output ./example/output` as an example, the command-line demos usually generate the following:

- `./example/output/Annotations`: 每一帧对应的分割 mask PNG / Per-frame segmentation mask PNG files
- `./example/output/Visualizations`: 叠加可视化结果 / Overlay visualizations
- `./example/output/pred.json`: 视频级别的结果摘要和每帧 `segments_info` / Video-level result summary with per-frame `segments_info`

中文：如果你接入的是自己的帧目录，只需要把 `--img_path` 指向该目录即可，输出结构保持一致。

English: If you use your own frame directory, simply point `--img_path` to that folder; the output structure stays the same.

## 7. 常见注意事项 / Notes and troubleshooting

- 中文：该仓库 README 明确说明主要在 Ubuntu 上测试；其他系统请预留额外适配成本。
  English: The main README explicitly says the project is tested on Ubuntu; expect extra setup work on other systems.
- 中文：文本提示模式依赖 Grounded-Segment-Anything；如果相关依赖未正确安装，文本模式和部分 Gradio 功能都可能无法正常启动。
  English: Text-prompted mode depends on Grounded-Segment-Anything; if that stack is not installed correctly, text mode and parts of the Gradio workflow may fail to start.
- 中文：推理速度与显存占用会明显受到 `--chunk_size`、`--size`、`--detection_every` 和场景中的目标数量影响。
  English: Runtime and memory usage are strongly affected by `--chunk_size`, `--size`, `--detection_every`, and the number of objects in the scene.
- 中文：如果你想进一步了解参数含义和加速建议，可以继续阅读 [DEMO.md](./DEMO.md)。
  English: For more argument details and performance tips, continue with [DEMO.md](./DEMO.md).
