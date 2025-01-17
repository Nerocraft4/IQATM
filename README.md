# Deep **T**one **M**apping Opetator Using **I**mage **Q**uality **A**ssessment Inspired Semi-supervised Training (IQATM)
This is the official Tensorflow implementation of the paper  
[C. Guo and X. Jiang, "Deep **T**one-**M**apping Operator Using **I**mage **Q**uality **A**ssessment Inspired Semi-Supervised Learning," in IEEE Access, vol. 9, pp. 73873-73889, 2021](https://ieeexplore.ieee.org/document/9431092).  
<div align=center><img src="https://ieeexplore.ieee.org/ielx7/6287639/9312710/9431092/graphical_abstract/access-gagraphic-3080331.jpg" width="500"></div>  

## Introduction
Tone-mapping is used to display HDR (High Dynamic Range) image on a traditional SDR/LDR (Standrad Dynamic Range, a.k.a. Low Dynamic Range, LDR) display. Its result is usually stored as an SDR image. That's to say, tone-mapping is the reverse process of single-shot HDR image generation (a.k.a. reverse/inverse tone-mapping, and SI-HDR).  
There are 2 types of HDR content: photometrically **linear** HDR, which is used in photograhpy, medicine and image based lighting, and **non-linear** HDRTV content with PQ/HLG non-linearity and wide gamut RGB primaries, which is used in film and television. Dynamic range of the HDR content is relatively higher than HDRTV.
Specifically, our work deals with **linear** HDR content. Checkpoint for **non-linear** HDRTV content has not been trained yet.
## Prerequisites
+ Unbuntu with PyCharm IDE
+ Python 2.7
+ CUDA 8.0 & CuDNN 6.0.21
+ Tensorflow-GPU 1.4.1
+ Other packages: opencv-python, imageio, easydict, etc.
## How to test
#### 1. Downloading checkpoint
Download checkpoint (model parameters) from [BaiduYunNetDisk](https://pan.baidu.com/s/1NCIFLaEuPNlJK2B5QkT5aw) (password: 9yvv) or [GoogleDrive](https://drive.google.com/drive/folders/1qDF8gR79JiTY8bBXvtU3q5JniKHAMmre?usp=sharing), make sure checkpoint (3 files suffixed `.data-00000-of-00001`, `.index` and `.meta` respectively) and a `checkpoint` file indicating the index of checkpoint are placed under `/checkpoint/ftlayer`.
#### 2. Preparing data
Place your testing HDR images under `/dataset/test` folder. We recommend to use `.hdr` encapsulation, otherwise you have to go to `/utils/configs.py` and change `config.data.appendix_hdr` to your one as long as package `imageio` supports it.
#### 3. Generating TFRecord
Run `/generate_tfrec.py`, note that our program will automatically clip some boundary pixels if image hight or width could not be divided by 8. (Optional) you can set `test_resize_to_half = True` if you later find the GPU runs out of memory.
#### 4. Testing
Run `/test.py`, results will be stored under `/result` folder.
## How to Train
#### 1. (Optional) Downloading pre-trained loss network
If your want to use "preceptual loss ***l<sub>p</sub>***", download `vgg16.npy` [here](https://mega.nz/#!YU1FWJrA!O1ywiCS2IiOlUCtCpI6HTJOMrneN-Qdv3ywQP5poecM) and place it under `/loss/pretrained`, clone [Pre-trained VGG-16](https://github.com/machrisaa/tensorflow-vgg) and place this repository under `/loss/`.
#### 2. Preparing data
Place your HDR and SDR images under `/dataset/train/hdr` and `/dataset/train/sdr`, respectively. Run `/generate_tfrec.py` with `phase = 'training'` & `ft = False` to generate TFRcord for the separate training of 2 netwoek branches (***N<sub>G</sub>*** and ***N<sub>L</sub>***, i.e. **Step 1**); run `/generate_tfrec.py` with `phase = 'training'` & `ft = True` to generate TFRcord for the joint training of the whole network (**Step 2**).
#### 3. Strat training
Go to `/utils/configs.py` and change `config.train.train_set_size` to that of your training pair, and run files blow with specific value of `epochs`
|file to run|function|saved checkpoint|
|---|---|---|
|`/train_bot.py`|**Step 1**, training ***N<sub>G</sub>***|under `/checkpoint/botlayer`|
|`/train_high.py`|**Step 1**, training ***N<sub>L</sub>***|under `/checkpoint/highlayer`|
|`/train_ft.py`|**Step 2**, training the whole network|under `/checkpoint/ftlayer`|

+ Program will only save the checkpoint of 5 latest epochs, and they will be rewritten if training is not stoped.
+ You can skip **Step 1** at the cost of potential increase of training difficulty.
+ Training can be monitored using TensorBoard by runing `/run_tfboard.py`.
## Acknowledgments
We took [Deep Reformulated Laplacian Tone Mapping (DRLTM)](https://github.com/linmc86/Deep-Reformulated-Laplacian-Tone-Mapping) as the Tensorflow prototype at the early stage of our development, this greatly simplified our coding since we are not experienced in computer science.
