# Fast Vision Transformers  with HiLo Attention👋(NeurIPS 2022 Spotlight)
[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0) 
<a href="https://pytorch.org/get-started/locally/"><img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-ee4c2c?logo=pytorch&logoColor=white"></a>

This is the official PyTorch implementation of [Fast Vision Transformers with HiLo Attention](https://arxiv.org/abs/2205.13213).

By [Zizheng Pan](https://scholar.google.com.au/citations?user=w_VMopoAAAAJ&hl=en), [Jianfei Cai](https://scholar.google.com/citations?user=N6czCoUAAAAJ&hl=en), and [Bohan Zhuang](https://scholar.google.com.au/citations?user=DFuDBBwAAAAJ).

## News

- **17/11/2023**. Update detection scripts with mmdet v3.2.0.

- **20/04/2023**. Update training scripts with PyTorch 2.0. Support ONNX and TensorRT model conversion, see [here](https://github.com/ziplab/LITv2/blob/main/classification/README.md#onnx-and-tensorrt-model-conversion).

- **15/12/2022**. Releasing ImageNet pretrained weights of using different values of alpha.

- **11/11/2022.** LITv2 will be presented as Spotlight!

- **13/10/2022.** Update code for higher version of timm. Compatible with PyTorch 1.12.1 + CUDA 11.3 + timm 0.6.11.

- **30/09/2022.** Add [benchmarking results](https://github.com/ziplab/LITv2#single-attention-layer-benchmark) for single attention layer. HiLo is super fast on both CPU and GPU!
  
- **15/09/2022.** LITv2 is accepted by NeurIPS 2022! 🔥🔥🔥
  
- **16/06/2022.** We release the source code for classification/detection/segmentation, along with the pretrained weights. Any issues are welcomed!


## A Gentle Introduction

![hilo](.github/arch.png)


We introduce LITv2, a simple and effective ViT which performs favourably against the existing state-of-the-art methods across a spectrum of different model sizes with faster speed.

![hilo](.github/hilo.png)

The core of LITv2: **HiLo attention** HiLo is inspired by the insight that high frequencies in an image capture local fine details and low frequencies focus on global structures, whereas a multi-head self-attention layer neglects the characteristic of different frequencies. Therefore, we propose to disentangle the high/low frequency patterns in an attention layer by separating the heads into two groups, where one group encodes high frequencies via self-attention within each local window, and another group performs the attention to model the global relationship between the average-pooled low-frequency keys from each window and each query position in the input feature map. 

### A Simple Demo
To quickly understand HiLo attention, you only need to install PyTorch and try the following code in the root directory of this repo.

```python
from hilo import HiLo
import torch

model = HiLo(dim=384, num_heads=12, window_size=2, alpha=0.5)

x = torch.randn(64, 196, 384) # batch_size x num_tokens x hidden_dimension
out = model(x, 14, 14)
print(out.shape)
print(model.flops(14, 14)) # the numeber of FLOPs
```

Output:

```bash
torch.Size([64, 196, 384])
83467776
```



## Installation

### Requirements

- Linux with Python ≥ 3.6
- PyTorch >= 1.8.1
- timm >= 0.3.2
- CUDA 11.1
- An NVIDIA GPU

### Conda environment setup

**Note**: You can use the same environment to debug [LITv1](https://github.com/ziplab/LIT). Otherwise, you can create a new python virtual environment by the following script.

```bash
conda create -n lit python=3.7
conda activate lit

# Install Pytorch and TorchVision
pip install torch==1.8.1+cu111 torchvision==0.9.1+cu111 torchaudio==0.8.1 -f https://download.pytorch.org/whl/torch_stable.html

pip install timm
pip install ninja
pip install tensorboard

# Install NVIDIA apex
git clone https://github.com/NVIDIA/apex
cd apex
pip install -v --disable-pip-version-check --no-cache-dir --global-option="--cpp_ext" --global-option="--cuda_ext" ./
cd ../
rm -rf apex/

# Build Deformable Convolution
cd mm_modules/DCN
python setup.py build install

pip install opencv-python==4.4.0.46 termcolor==1.1.0 yacs==0.1.8
```



# Getting Started

For image classification on ImageNet, please refer to [classification](https://github.com/ziplab/LITv2/tree/main/classification).

For object detection on COCO 2017, please refer to [detection](https://github.com/ziplab/LITv2/tree/main/detection).

For semantic segmentation on ADE20K, please refer to [segmentation](https://github.com/ziplab/LITv2/tree/main/segmentation).



## Results and Model Zoo

**Note:** For your convenience, you can find all models and logs from [Google Drive](https://drive.google.com/drive/folders/1VAtrPWEqxi-6q6luwEVdYvkBYedApwbU?usp=sharing) (4.8G in total). Alternatively, we also provide download links from github.

### Image Classification on ImageNet-1K

All models are trained with 300 epochs with a total batch size of 1024 on 8 V100 GPUs.

| Model   | Resolution | Params (M) | FLOPs (G) | Throughput (imgs/s) | Train Mem (GB) | Test Mem (GB) | Top-1 (%) | Download                                                     |
| ------- | ---------- | ---------- | --------- | ------------------- | -------------- | ------------- | --------- | ------------------------------------------------------------ |
| LITv2-S | 224        | 28         | 3.7       | 1,471               | 5.1            | 1.2           | 82.0      | [model](https://github.com/ziplab/LITv2/releases/download/v1.0/litv2_s.pth) & [log](https://github.com/ziplab/LITv2/releases/download/v1.0/litv2_s_log.txt) |
| LITv2-M | 224        | 49         | 7.5       | 812                 | 8.8            | 1.4           | 83.3      | [model](https://github.com/ziplab/LITv2/releases/download/v1.0/litv2_m.pth) & [log](https://github.com/ziplab/LITv2/releases/download/v1.0/litv2_m_log.txt) |
| LITv2-B | 224        | 87         | 13.2      | 602                 | 12.2           | 2.1           | 83.6      | [model](https://github.com/ziplab/LITv2/releases/download/v1.0/litv2_b.pth) & [log](https://github.com/ziplab/LITv2/releases/download/v1.0/litv2_b_log.txt) |
| LITv2-B | 384        | 87         | 39.7      | 198                 | 35.8           | 4.6           | 84.7      | [model](https://github.com/ziplab/LITv2/releases/download/v1.0/litv2_b_384.pth) |

> By default, the throughput and memory footprint are tested on one RTX 3090 based on a batch size of 64. Memory is measured by the peak memory usage with `torch.cuda.max_memory_allocated()`. Throughput is averaged over 30 runs.



### Pretrained LITv2-S with Different Values of Alpha

| Alpha | Params (M) | Lo-Fi Heads | Hi-Fi Heads | FLOPs (G) | ImageNet Top1 (%) | Download                                                     |
| ----- | ---------- | ----------- | ----------- | --------- | ----------------- | ------------------------------------------------------------ |
| 0.0   | 28         | 0           | 12          | 3.97      | 81.16             | [github](https://github.com/ziplab/LITv2/releases/download/v1.1/alpha_0.pth) |
| 0.2   | 28         | 2           | 10          | 3.88      | 81.89             | [github](https://github.com/ziplab/LITv2/releases/download/v1.1/alpha_0.2.pth) |
| 0.4   | 28         | 4           | 8           | 3.82      | 81.81             | [github](https://github.com/ziplab/LITv2/releases/download/v1.1/alpha_0.4.pth) |
| 0.5   | 28         | 6           | 6           | 3.77      | 81.88             | [github](https://github.com/ziplab/LITv2/releases/download/v1.1/alpha_0.5.pth) |
| 0.7   | 28         | 8           | 4           | 3.74      | 81.94             | [github](https://github.com/ziplab/LITv2/releases/download/v1.1/alpha_0.7.pth) |
| 0.9   | 28         | 10          | 2           | 3.73      | 82.03             | [github](https://github.com/ziplab/LITv2/releases/download/v1.1/alpha_0.9.pth) |
| 1.0   | 28         | 12          | 0           | 3.70      | 81.89             | [github](https://github.com/ziplab/LITv2/releases/download/v1.1/alpha_1.pth) |

> Pretrained weights from the experiments of Figure 4: Effect of α based on LITv2-S.



### Object Detection on COCO 2017

All models are trained with 1x schedule (12 epochs) with a total batch size of 16 on 8 V100 GPUs.

#### RetinaNet

| Backbone | Window Size | Params (M) | FLOPs (G) | FPS  | box AP | Config                                                       | Download                                                     |
| -------- | ----------- | ---------- | --------- | ---- | ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| LITv2-S  | 2           | 38         | 242       | 18.7 | 44.0   | [config](https://github.com/ziplab/LITv2/blob/main/detection/configs/litv2/retinanet_litv2_s_fpn_1x_coco.py) | [model](https://github.com/ziplab/LITv2/releases/download/v1.0/retinanet_litv2_s_fpn_1x_coco.pth) & [log](https://github.com/ziplab/LITv2/releases/download/v1.0/retinanet_litv2_s_fpn_1x_coco_log.json) |
| LITv2-S  | 4           | 38         | 230       | 20.4 | 43.7   | [config](https://github.com/ziplab/LITv2/blob/main/detection/configs/litv2/retinanet_litv2_s_fpn_1x_coco_ws_4.py) | [model](https://github.com/ziplab/LITv2/releases/download/v1.0/retinanet_litv2_s_fpn_1x_coco_ws_4.pth) & [log](https://github.com/ziplab/LITv2/releases/download/v1.0/retinanet_litv2_s_fpn_1x_coco_ws_4_log.json) |
| LITv2-M  | 2           | 59         | 348       | 12.2 | 46.0   | [config](https://github.com/ziplab/LITv2/blob/main/detection/configs/litv2/retinanet_litv2_m_fpn_1x_coco.py) | [model](https://github.com/ziplab/LITv2/releases/download/v1.0/retinanet_litv2_m_fpn_1x_coco.pth) & [log](https://github.com/ziplab/LITv2/releases/download/v1.0/retinanet_litv2_m_fpn_1x_coco_log.json) |
| LITv2-M  | 4           | 59         | 312       | 14.8 | 45.8   | [config](https://github.com/ziplab/LITv2/blob/main/detection/configs/litv2/retinanet_litv2_m_fpn_1x_coco_ws_4.py) | [model](https://github.com/ziplab/LITv2/releases/download/v1.0/retinanet_litv2_m_fpn_1x_coco_ws_4.pth) & [log](https://github.com/ziplab/LITv2/releases/download/v1.0/retinanet_litv2_m_fpn_1x_coco_ws_4_log.json) |
| LITv2-B  | 2           | 97         | 481       | 9.5  | 46.7   | [config](https://github.com/ziplab/LITv2/blob/main/detection/configs/litv2/retinanet_litv2_b_fpn_1x_coco.py) | [model](https://github.com/ziplab/LITv2/releases/download/v1.0/retinanet_litv2_b_fpn_1x_coco.pth) & [log](https://github.com/ziplab/LITv2/releases/download/v1.0/retinanet_litv2_b_fpn_1x_coco_log.json) |
| LITv2-B  | 4           | 97         | 430       | 11.8 | 46.3   | [config](https://github.com/ziplab/LITv2/blob/main/detection/configs/litv2/retinanet_litv2_b_fpn_1x_coco_ws_4.py) | [model](https://github.com/ziplab/LITv2/releases/download/v1.0/retinanet_litv2_b_fpn_1x_coco_ws_4.pth) & [log](https://github.com/ziplab/LITv2/releases/download/v1.0/retinanet_litv2_b_fpn_1x_coco_ws_4_log.json) |

#### Mask R-CNN

| Backbone | Window Size | Params (M) | FLOPs (G) | FPS  | box AP | mask AP | Config                                                       | Download                                                     |
| -------- | ----------- | ---------- | --------- | ---- | ------ | ------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| LITv2-S  | 2           | 47         | 261       | 18.7 | 44.9   | 40.8    | [config](https://github.com/ziplab/LITv2/blob/main/detection/configs/litv2/mask_rcnn_litv2_s_fpn_1x_coco.py) | [model](https://github.com/ziplab/LITv2/releases/download/v1.0/mask_rcnn_litv2_s_fpn_1x_coco.pth) & [log](https://github.com/ziplab/LITv2/releases/download/v1.0/mask_rcnn_litv2_s_fpn_1x_coco_log.json) |
| LITv2-S  | 4           | 47         | 249       | 21.9 | 44.7   | 40.7    | [config](https://github.com/ziplab/LITv2/blob/main/detection/configs/litv2/mask_rcnn_litv2_s_fpn_1x_coco_ws_4.py) | [model](https://github.com/ziplab/LITv2/releases/download/v1.0/mask_rcnn_litv2_s_fpn_1x_coco_ws_4.pth) & [log](https://github.com/ziplab/LITv2/releases/download/v1.0/mask_rcnn_litv2_s_fpn_1x_coco_ws_4_log.json) |
| LITv2-M  | 2           | 68         | 367       | 12.6 | 46.8   | 42.3    | [config](https://github.com/ziplab/LITv2/blob/main/detection/configs/litv2/mask_rcnn_litv2_m_fpn_1x_coco.py) | [model](https://github.com/ziplab/LITv2/releases/download/v1.0/mask_rcnn_litv2_m_fpn_1x_coco.pth) & [log](https://github.com/ziplab/LITv2/releases/download/v1.0/mask_rcnn_litv2_m_fpn_1x_coco_log.json) |
| LITv2-M  | 4           | 68         | 315       | 16.0 | 46.5   | 42.0    | [config](https://github.com/ziplab/LITv2/blob/main/detection/configs/litv2/mask_rcnn_litv2_m_fpn_1x_coco_ws_4.py) | [model](https://github.com/ziplab/LITv2/releases/download/v1.0/mask_rcnn_litv2_m_fpn_1x_coco_ws_4.pth) & [log](https://github.com/ziplab/LITv2/releases/download/v1.0/mask_rcnn_litv2_m_fpn_1x_coco_ws_4_log.json) |
| LITv2-B  | 2           | 106        | 500       | 9.3  | 47.3   | 42.6    | [config](https://github.com/ziplab/LITv2/blob/main/detection/configs/litv2/mask_rcnn_litv2_b_fpn_1x_coco.py) | [model](https://github.com/ziplab/LITv2/releases/download/v1.0/mask_rcnn_litv2_b_fpn_1x_coco.pth) & [log](https://github.com/ziplab/LITv2/releases/download/v1.0/mask_rcnn_litv2_b_fpn_1x_coco_log.json) |
| LITv2-B  | 4           | 106        | 449       | 11.5 | 46.8   | 42.3    | [config](https://github.com/ziplab/LITv2/blob/main/detection/configs/litv2/mask_rcnn_litv2_b_fpn_1x_coco_ws_4.py) | [model](https://github.com/ziplab/LITv2/releases/download/v1.0/mask_rcnn_litv2_b_fpn_1x_coco_ws_4.pth) & [log](https://github.com/ziplab/LITv2/releases/download/v1.0/mask_rcnn_litv2_b_fpn_1x_coco_ws_4_log.json) |

### Semantic Segmentation on ADE20K

All models are trained with 80K iterations with a total batch size of 16 on 8 V100 GPUs.

| Backbone | Params (M) | FLOPs (G) | FPS  | mIoU | Config                                                       | Download                                                     |
| -------- | ---------- | --------- | ---- | ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| LITv2-S  | 31         | 41        | 42.6 | 44.3 | [config](https://github.com/ziplab/LITv2/blob/main/segmentation/configs/litv2/litv2_s_fpn_r50_512x512_80k_ade20k.py) | [model](https://github.com/ziplab/LITv2/releases/download/v1.0/litv2_s_fpn_r50_512x512_80k_ade20k.pth) & [log](https://github.com/ziplab/LITv2/releases/download/v1.0/litv2_s_fpn_r50_512x512_80k_ade20k_log.json) |
| LITv2-M  | 52         | 63        | 28.5 | 45.7 | [config](https://github.com/ziplab/LITv2/blob/main/segmentation/configs/litv2/litv2_m_fpn_r50_512x512_80k_ade20k.py) | [model](https://github.com/ziplab/LITv2/releases/download/v1.0/litv2_m_fpn_r50_512x512_80k_ade20k.pth) & [log](https://github.com/ziplab/LITv2/releases/download/v1.0/litv2_m_fpn_r50_512x512_80k_ade20k_log.json) |
| LITv2-B  | 90         | 93        | 27.5 | 47.2 | [config](https://github.com/ziplab/LITv2/blob/main/segmentation/configs/litv2/litv2_b_fpn_r50_512x512_80k_ade20k.py) | [model](https://github.com/ziplab/LITv2/releases/download/v1.0/litv2_b_fpn_r50_512x512_80k_ade20k.pth) & [log](https://github.com/ziplab/LITv2/releases/download/v1.0/litv2_b_fpn_r50_512x512_80k_ade20k_log.json) |


### Benchmarking Throughput on More GPUs

| Model         | Params (M) | FLOPs (G) | A100      | V100      | RTX 6000 | RTX 3090  | Top-1 (%) |
| ------------- | ---------- | --------- | --------- | --------- | -------- | --------- | --------- |
| ResNet-50     | 26         | 4.1       | 1,424     | 1,123     | 877      | 1,279     | 80.4      |
| PVT-S         | 25         | 3.8       | 1,460     | 798       | 548      | 1,007     | 79.8      |
| Twins-PCPVT-S | 24         | 3.8       | 1,455     | 792       | 529      | 998       | 81.2      |
| Swin-Ti       | 28         | 4.5       | 1,564     | 1,039     | 710      | 961       | 81.3      |
| TNT-S         | 24         | 5.2       | 802       | 431       | 298      | 534       | 81.3      |
| CvT-13        | 20         | 4.5       | 1,595     | 716       | 379      | 947       | 81.6      |
| CoAtNet-0     | 25         | 4.2       | 1,538     | 962       | 643      | 1,151     | 81.6      |
| CaiT-XS24     | 27         | 5.4       | 991       | 484       | 299      | 623       | 81.8      |
| PVTv2-B2      | 25         | 4.0       | 1,175     | 670       | 451      | 854       | 82.0      |
| XCiT-S12      | 26         | 4.8       | 1,727     | 761       | 504      | 1,068     | 82.0      |
| ConvNext-Ti   | 28         | 4.5       | 1,654     | 762       | 571      | 1,079     | 82.1      |
| Focal-Tiny    | 29         | 4.9       | 471       | 372       | 261      | 384       | **82.2**  |
| **LITv2-S**       | 28         | **3.7**   | **1,874** | **1,304** | **928**  | **1,471** | 82.0      |



### Single Attention Layer Benchmark

The following visualization results can refer to [vit-attention-benchmark](https://github.com/HubHop/vit-attention-benchmark).

![hilo_cpu_gpu](.github/hilo_cpu_gpu.png)


## Citation

If you use LITv2 in your research, please consider the following BibTeX entry and giving us a star 🌟.

```BibTeX
@inproceedings{pan2022hilo,
  title={Fast Vision Transformers with HiLo Attention},
  author={Pan, Zizheng and Cai, Jianfei and Zhuang, Bohan},
  booktitle={NeurIPS},
  year={2022}
}
```

If you find the code useful, please also consider the following BibTeX entry

```BibTeX
@inproceedings{pan2022litv1,
  title={Less is More: Pay Less Attention in Vision Transformers},
  author={Pan, Zizheng and Zhuang, Bohan and He, Haoyu and Liu, Jing and Cai, Jianfei},
  booktitle={AAAI},
  year={2022}
}
```



## License

This repository is released under the Apache 2.0 license as found in the [LICENSE](https://github.com/ziplab/LITv2/blob/main/LICENSE) file.



## Acknowledgement

This repository is built upon [DeiT](https://github.com/facebookresearch/deit), [Swin](https://github.com/microsoft/Swin-Transformer) and [LIT](https://github.com/ziplab/LIT), we thank the authors for their open-sourced code.

