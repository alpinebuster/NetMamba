<div align="center">
<h1>NetMamba </h1>
<h3>Efficient Network Traffic Classification via Pre-training Unidirectional Mamba</h3>

[Tongze Wang](https://github.com/wangtz19), [Xiaohui Xie](https://thuxiexiaohui.github.io/), [Wenduo Wang](https://github.com/Viz7), [Chuyi Wang](https://github.com/Judy456abc), [Youjian Zhao](https://www.cs.tsinghua.edu.cn/info/1126/3576.htm), [Yong Cui](https://www.cuiyong.net/index.html)

ICNP 2024 ([arXiv paper](https://arxiv.org/abs/2405.11449))
</div>

## Overview
<div align="center">
<img src="assets/NetMamba.png" />
</div>

## Updates
[2026-04-05] We have released the source code for our journal paper [NetMamba+](https://arxiv.org/abs/2601.21792). Click [here](https://github.com/wangtz19/NetMambaPlus) and try the latest version.

## Environment Setup
- Create python environment
    - `pyenv install 3.10.13`
    - `pyenv virtualenv 3.10.13 netmamba`
    - `pyenv activate netmamba`
- Install PyTorch 2.1.1+cu121 (we conduct experiments on this version)
    - `pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple`
    - `pip install torch==2.1.1 torchvision==0.16.1 --index-url https://download.pytorch.org/whl/cu121`
- Install Mamba 1.1.1
    - `cd mamba-1p1p1`
    - `export CUDA_HOME=/usr/local/cuda-12`
    - `export PATH=$CUDA_HOME/bin:$PATH`
    - `export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$LD_LIBRARY_PATH`
    - `pip install packaging wheel`
    - `pip install --no-build-isolation -e .`
- Install other dependent libraries
    - `pip install -r requirements.txt --no-build-isolation`

## Data Preparation
### Download our processed datasets
For simplicity, you are welcome to download our processed datasets on which our experiments are conducted from [google drive](https://drive.google.com/drive/folders/1C1urXBhk09V7Z80Kk5JYuP7QeXiedUIl?usp=sharing). 

Each dataset is organized into the following structure:
```text
.
|-- train
|   |-- Category 1
|   |   |-- Sample 1
|   |   |-- Sample 2
|   |   |-- ...
|   |   `-- Sample M
|   |-- Category 2
|   |-- ...
|   `-- Catergory N
|-- test
`-- valid
```
### Process your own datasets
If you'd like to generate customized datasets, please refer to preprocessing scripts provided in [dataset](https://github.com/wangtz19/NetMamba/tree/main/dataset). Note that you need to change several file paths accordingly.

## Run NetMamba
- Run pre-training: 
```shell
CUDA_VISIBLE_DEVICES=0 python src/pre-train.py \\
    --batch_size 128 \\
    --blr 1e-3 \\
    --steps 150000 \\
    --mask_ratio 0.9 \\
    --data_path <your-dataset-dir> \\
    --output_dir <your-output-dir> \\
    --log_dir <your-output-dir> \\
    --model net_mamba_pretrain \\
    --no_amp
```
- Run fine-tuning (including evaluation)
```shell
CUDA_VISIBLE_DEVICES=0 python src/fine-tune.py \\
    --blr 2e-3 \\
    --epochs 120 \\
    --nb_classes <num-class> \\
    --finetune <pretrain-checkpoint-path> \\
    --data_path <your-dataset-dir> \\
    --output_dir <your-output-dir> \\
    --log_dir <your-output-dir> \\
    --model net_mamba_classifier \\
    --no_amp


# cicdarknet
CUDA_VISIBLE_DEVICES=0 nohup python src/fine-tune.py \
    --blr 2e-3 \
    --epochs 120 \
    --nb_classes 7 \
    --finetune ./pre-train.pth \
    --data_path ./dataset/cicdarknet/dataset_sampled \
    --output_dir ./dataset/cicdarknet/output_ratio1.0 \
    --log_dir dataset/cicdarknet/logs_ratio1.0 \
    --model net_mamba_classifier \
    --no_amp \
> cicdarknet_ratio1.0.log 2>&1 &

CUDA_VISIBLE_DEVICES=0 nohup python src/fine-tune.py \
    --blr 2e-3 \
    --epochs 120 \
    --nb_classes 7 \
    --finetune ./pre-train.pth \
    --data_path ./dataset/cicdarknet/dataset_sampled_ratio0.001 \
    --output_dir ./dataset/cicdarknet/output_ratio0.001 \
    --log_dir dataset/cicdarknet/logs_ratio0.001 \
    --model net_mamba_classifier \
    --no_amp \
> cicdarknet_ratio0.001.log 2>&1 &

CUDA_VISIBLE_DEVICES=1 nohup python src/fine-tune.py \
    --blr 2e-3 \
    --epochs 120 \
    --nb_classes 7 \
    --batch_size 32 \
    --finetune ./pre-train.pth \
    --data_path ./dataset/cicdarknet/dataset_sampled_ratio1e-05 \
    --output_dir ./dataset/cicdarknet/output_ratio1e-05 \
    --log_dir dataset/cicdarknet/logs_ratio1e-05 \
    --model net_mamba_classifier \
    --no_amp \
> cicdarknet_ratio1e-05.log 2>&1 &


# tcub
CUDA_VISIBLE_DEVICES=1 nohup python src/fine-tune.py \
    --blr 2e-3 \
    --epochs 120 \
    --nb_classes 7 \
    --finetune ./pre-train.pth \
    --data_path ./dataset/tcub/dataset_sampled \
    --output_dir ./dataset/tcub/output_ratio1.0 \
    --log_dir dataset/tcub/logs_ratio1.0 \
    --model net_mamba_classifier \
    --no_amp \
> tcub_ratio1.0.log 2>&1 &

CUDA_VISIBLE_DEVICES=1 nohup python src/fine-tune.py \
    --blr 2e-3 \
    --epochs 120 \
    --nb_classes 7 \
    --finetune ./pre-train.pth \
    --data_path ./dataset/tcub/dataset_sampled_ratio0.001 \
    --output_dir ./dataset/tcub/output_ratio0.001 \
    --log_dir dataset/tcub/logs_ratio0.001 \
    --model net_mamba_classifier \
    --no_amp \
> tcub_ratio0.001.log 2>&1 &

CUDA_VISIBLE_DEVICES=1 nohup python src/fine-tune.py \
    --blr 2e-3 \
    --epochs 120 \
    --nb_classes 7 \
    --batch_size 16 \
    --finetune ./pre-train.pth \
    --data_path ./dataset/tcub/dataset_sampled_ratio1e-05 \
    --output_dir ./dataset/tcub/output_ratio1e-05 \
    --log_dir dataset/tcub/logs_ratio1e-05 \
    --model net_mamba_classifier \
    --no_amp \
> tcub_ratio1e-05.log 2>&1 &


# safesurf2025
CUDA_VISIBLE_DEVICES=1 nohup python src/fine-tune.py \
    --blr 2e-3 \
    --epochs 120 \
    --nb_classes 7 \
    --finetune ./pre-train.pth \
    --data_path ./dataset/safesurf2025/dataset_sampled \
    --output_dir ./dataset/safesurf2025/output_ratio1.0 \
    --log_dir dataset/safesurf2025/logs_ratio1.0 \
    --model net_mamba_classifier \
    --no_amp \
> safesurf2025_ratio1.0.log 2>&1 &

CUDA_VISIBLE_DEVICES=0 nohup python src/fine-tune.py \
    --blr 2e-3 \
    --epochs 120 \
    --nb_classes 7 \
    --finetune ./pre-train.pth \
    --data_path ./dataset/safesurf2025/dataset_sampled_ratio0.001 \
    --output_dir ./dataset/safesurf2025/output_ratio0.001 \
    --log_dir dataset/safesurf2025/logs_ratio0.001 \
    --model net_mamba_classifier \
    --no_amp \
> safesurf2025_ratio0.001.log 2>&1 &

CUDA_VISIBLE_DEVICES=1 nohup python src/fine-tune.py \
    --blr 2e-3 \
    --epochs 120 \
    --nb_classes 7 \
    --batch_size 32 \
    --finetune ./pre-train.pth \
    --data_path ./dataset/safesurf2025/dataset_sampled_ratio1e-05 \
    --output_dir ./dataset/safesurf2025/output_ratio1e-05 \
    --log_dir dataset/safesurf2025/logs_ratio1e-05 \
    --model net_mamba_classifier \
    --no_amp \
> safesurf2025_ratio1e-05.log 2>&1 &
```
Note that you should replace variable in the `< >` format with your actual values.

## Checkpoint
The pre-trained checkpoint of NetMamba is available for download on our [huggingface repo](https://huggingface.co/wangtz/NetMamba). Feel free to access it at your convenience. If you require any other type of checkpoints, please contact us via email (wangtz23@mails.tsinghua.edu.cn).

## Citation
```
@inproceedings{wang2024netmamba,
  title={Netmamba: Efficient network traffic classification via pre-training unidirectional mamba},
  author={Wang, Tongze and Xie, Xiaohui and Wang, Wenduo and Wang, Chuyi and Zhao, Youjian and Cui, Yong},
  booktitle={2024 IEEE 32nd International Conference on Network Protocols (ICNP)},
  pages={1--11},
  year={2024},
  organization={IEEE}
}
```