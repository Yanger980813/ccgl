# CCGL: Contrastive Cascade Graph Learning

![](https://img.shields.io/badge/TKDE-22-blue)
![](https://img.shields.io/badge/python-3.7-green)
![](https://img.shields.io/badge/tensorflow-2.3-green)
![](https://img.shields.io/badge/cudatoolkit-10.1-green)
![](https://img.shields.io/badge/cudnn-7.6.5-green)

This repo provides a reference implementation of Contrastive Cascade Graph Learning (**CCGL**) framework as described in the paper:

> CCGL: Contrastive Cascade Graph Learning   
> [Xovee Xu](https://www.xoveexu.com), [Fan Zhou](https://dblp.org/pid/63/3122-2.html), [Kunpeng Zhang](http://www.terpconnect.umd.edu/~kpzhang/), and [Siyuan Liu](https://directory.smeal.psu.edu/sxl68)  
> IEEE Transactions on Knowledge and Data Engineering (TKDE), vol. 35, no. 5, pp. 4539--4554, May 2023   
> [arXiv:2107.12576](https://arxiv.org/abs/2107.12576)  
> https://doi.org/10.1109/TKDE.2022.3151829  


## Dataset

You can download all five datasets (Weibo, Twitter, ACM, APS, and DBLP) via any one of the following links:

Google Drive|Dropbox|Onedrive|Baidu Netdisk
:---:|:---:|:---:|:---:
<a href='https://drive.google.com/file/d/1wmUa7hvJlF5oCLVJ72OgyKnVkHZJX8jX/view?usp=sharing' target='_black'><img src='./.assets/200px-Google_Drive_logo.png' height=30px>|<a href='https://www.dropbox.com/s/0kadkjyuwffcuw2/datasets.zip?dl=0' target='_black'><img src='./.assets/140px-Microsoft_Office_OneDrive_(2018–present).png' height=30px></a>|<a href='https://1drv.ms/u/s!AsVLooK4NjBruTngZWgx1p0psD1k?e=5iMcVB' target='_black'><img src='.assets/dropbox.png' height=30px></a>|<a href='https://pan.baidu.com/s/16A5SvCVEtPYxelyYIlYAQg' target='_black'><img src='./.assets/baidu-netdisk.jpg' height=30px> `trqg`</a>


## Environmental Settings

Our experiments are conducted on Ubuntu 20.04, a single NVIDIA 1080Ti GPU, 48GB RAM, and Intel i7 8700K. CCGL is implemented by `Python 3.7`, `TensorFlow 2.3`, `Cuda 10.1`, and `Cudnn 7.6.5`.

Create a virtual environment and install GPU-support packages via [Anaconda](https://www.anaconda.com/):
```shell
# create virtual environment
conda create --name=ccgl python=3.7 cudatoolkit=10.1 cudnn=7.6.5

# activate virtual environment
conda activate ccgl

# install other dependencies
pip install -r requirements.txt
```

## Usage

Here we take Weibo dataset as an example to demonstrate the usage.

### Preprocess

Step 1: divide, filter, generate labeled and unlabeled cascades:
```shell
cd ccgl
# labeled cascades
python src/gene_cas.py --input=./datasets/weibo/ --unlabel=False
# unlabeled cascades
python src/gene_cas.py --input=./datasets/weibo/ --unlabel=True
```

Step 2: augment both labeled and unlabeled cascades (here we use the `AugSIM` strategy):
```shell
python src/augmentor.py --input=./datasets/weibo/ --aug_strategy=AugSIM
```

Step 3: generate cascade embeddings:
```shell
python src/gene_emb.py --input=./datasets/weibo/ 
```

### Pre-training

```shell
python src/pre_training.py --name=weibo-0 --input=./datasets/weibo/ --projection_head=4-1
```
The saved pre-training model is named as `weibo-0`. 

### Fine-tuning

```shell
python src/fine_tuning.py --name=weibo-0 --num=0 --input=./datasets/weibo/ --projection_head=4-1
```
Here we load the pre-trained model `weibo-0` and save the teacher network as `weibo-0-0`.

### Distillation

```shell
python src/distilling.py --name=weibo-0-0 --num=0 --input=./datasets/weibo/ --projection_head=4-1
```
Here we load the teacher network `weibo-0-0` and save the student network as `weibo-0-0-student-0`. 

### (Optional) Run the Base model

```shell
python src/base_model.py --input=./datasets/weibo/ 
```

## CCGL model weights

We provide pre-trained, fine-tuned, and distilled CCGL model weights. Please see details in the following table. 

Model|Dataset|Label Fraction|Projection Head|MSLE|Weights
:---|:---|:---|:---|:---|:---
Pre-trained CCGL model|Weibo|100%|4-1|-|[Download](./results/pre_training_weight/weibo-100.h5)
Pre-trained CCGL model|Weibo|10%|4-4|-|[Download](./results/pre_training_weight/weibo-10.h5)
Pre-trained CCGL model|Weibo|1%|4-3|-|[Download](./results/pre_training_weight/weibo-1.h5)
Fine-tuned CCGL model|Weibo|100%|4-1|2.70|[Download](./results/fine_tuning_weight/weibo-100-0.h5)
Fine-tuned CCGL model|Weibo|10%|4-4|2.87|[Download](./results/fine_tuning_weight/weibo-10-0.h5)
Fine-tuned CCGL model|Weibo|1%|4-3|3.30|[Download](./results/fine_tuning_weight/weibo-1-0.h5)

Load weights into the model:
```python
# construct model, carefully check projection head designs:
# use different number of Dense layers
...
# load weights for fine-tuning, distillation, or evaluation
model.load_weights(weight_path)
```
Check `src/fine_tuning.py` and `src/distilling.py` for *weights loading* examples.

## Default hyper-parameter settings

Unless otherwise specified, we use following default hyper-parameter settings.

Param|Value|Param|Value
:---|---:|:---|---:
Augmentation strength|0.1|Pre-training epochs|30
Augmentation strategy|AugSIM|Projection Head (100%)|4-1
Batch size|64|Projection Head (10%)|4-4
Early stopping patience|20|Projection Head (1%)|4-3
Embedding dimension|64|Model size|128 (4x)
Learning rate|5e-4|Temperature|0.1

## Change Logs

- Jul 21, 2021: fix a bug and some annotations

## Cite

If you find our paper & code are useful for your research, please consider citing us 😘:

```bibtex
@article{xu2022ccgl, 
  author = {Xovee Xu and Fan Zhou and Kunpeng Zhang and Siyuan Liu}, 
  title = {{CCGL}: Contrastive Cascade Graph Learning}, 
  journal = {IEEE Transactions on Knowledge and Data Engineering (TKDE)},
  volume = {35},
  number = {5},
  pages = {4539--4554},
  year = {2022},
  doi = {10.1109/TKDE.2022.3151829}, 
}
```

We also have a [survey paper](https://dl.acm.org/doi/10.1145/3433000?cid=99659687041) you might be interested:

```bibtex
@article{zhou2021survey,
  author = {Fan Zhou and Xovee Xu and Goce Trajcevski and Kunpeng Zhang}, 
  title = {A Survey of Information Cascade Analysis: Models, Predictions, and Recent Advances}, 
  journal = {ACM Computing Surveys (CSUR)}, 
  volume = {54},
  number = {2},
  year = {2021},
  articleno = {27},
  numpages = {36},
  doi = {10.1145/3433000},
}
```

## Acknowledgment

We would like to thank [Xiuxiu Qi](https://qhemu.github.io/xiuxiuqi/), [Ce Li](https://github.com/celi52), [Qing Yang](https://www.linkedin.com/in/庆-杨-43ba1a142), and Wenxiong Li for sharing their computing resources and help us to test the codes. We would also like to show our gratitude to the authors of [SimCLR](https://github.com/google-research/simclr) (and [Sayak Paul](https://github.com/sayakpaul)), [node2vec](https://github.com/eliorc/node2vec), [DeepHawkes](https://github.com/CaoQi92/DeepHawkes), and others, for sharing their codes and datasets. 

## Contact

For any questions please open an issue or drop an email to: `xovee at live.com`
