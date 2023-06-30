---
layout: post
title:  保姆级教程：如何本地部署私有chatglm模型及知识库扩展调试
date: 2023-06-30 9:22:11
tags:
- 技术
categories:
- [AIGC]
toc: true
description: 如何本地部署私有chatglm模型，以及可落地的应用
---

## 部署条件准备
* 一台有显卡的linux服务器，显存达到8G以上，内存达到8G以上，硬盘空间达到100G以上

## 开始部署

### 我的部署环境
* 系统：Ubuntu 16.04
* 显卡：NVIDIA GeForce GTX 1080 Ti
* 显存：11G
* 内存：64G
* 硬盘：1T

### 部署步骤

#### 1.检查你的python版本
```bash
python3 --version
```
建议3.8 - 3.10最佳，如果不是，建议安装conda，创建一个新的python环境
1.conda安装
```bash
wget https://repo.anaconda.com/archive/Anaconda3-2021.05-Linux-x86_64.sh
bash Anaconda3-2021.05-Linux-x86_64.sh
```
2.创建新环境及激活
```bash
conda create -n chatglm python=3.8
conda activate chatglm
```

#### 2.下载gitchain-chatglm代码
```bash
git clone https://github.com/imClumsyPanda/langchain-ChatGLM.git
```
#### 3.安装依赖
需要提前安装paddleocr依赖libX11，libXext

```bash
apt install libx11-dev libxext-dev libxtst-dev libxrender-dev libxmu-dev libxmuu-dev 
apt install libXext
```

然后安装python依赖：
```bash
cd langchain-ChatGLM
pip install -r requirements.txt
```
检查有无问题，看不懂的话，就不要管了，反正我也看不懂。
实在要管的话，就复制报错信息，发给chatGPT询问。


#### 4.下载模型
该项目使用了两种模型，一种是在上传知识库文件时，用来语义分段和向量转化的embedding模型,如text2vec，
一种是用来生成回复的LLM模型，如chatGLM。

如果你网络环境好，人在美国刚下飞机，可以不用手动下载，直接运行python webui.py，会自动下载模型
如果不好（大概率不好），请手动下载模型，注意这里未必需要全部下载，embedding模型和llm模型各下载一个就可以，
手动下载模型有两个方法：
1.使用百度网盘下载
  1. ernie-3.0-base-zh.zip [链接](https://pan.baidu.com/s/1CIvKnD3qzE-orFouA8qvNQ?pwd=4wih)
  2. ernie-3.0-nano-zh.zip 链接: https://pan.baidu.com/s/1Fh8fgzVdavf5P1omAJJ-Zw?pwd=q6s5
  3. text2vec-large-chinese.zip 链接: https://pan.baidu.com/s/1sMyPzBIXdEzHygftEoyBuA?pwd=4xs7
  4. chatglm-6b-int4-qe.zip 链接: https://pan.baidu.com/s/1DDKMOMHtNZccOOBGWIOYww?pwd=22ji
  5. chatglm-6b-int4.zip 链接: https://pan.baidu.com/s/1pvZ6pMzovjhkA6uPcRLuJA?pwd=3gjd
  6. chatglm-6b.zip 链接: https://pan.baidu.com/s/1B-MpsVVs1GHhteVBetaquw?pwd=djay

2.访问huggingface网页，找到相关模型项目，自己在电脑上建个文件夹，
然后在hugginnface的file目录里挨个点击下载（有时候huggingface的访问会不通，多试几次）
huggingface官网：https://huggingface.co/
llm模型（chatglm-6b-int4）：https://huggingface.co/THUDM/chatglm-6b-int4/tree/main
embedding模型（text2vec-large-chinese）：https://huggingface.co/THUDM/text2vec-large-chinese/tree/main


这个方法适用于chatglm2 和一些其他后期你想扩展使用的模型

下载完成后，上传并解压到服务器任一目录，然后修改config.py里的模型路径
```bash
cd langchain-ChatGLM
vim config/model_config.py
```

在config文件里需要修改两处
1.修改embedding模型路径，我的路径如下"/root/langchain/embedding/"，你的路径可能不同，需要修改
```python
# 在以下字典中修改属性值，以指定本地embedding模型存储位置
# 如将 "text2vec": "GanymedeNil/text2vec-large-chinese" 修改为 "text2vec": "User/Downloads/text2vec-large-chinese"
# 此处请写绝对路径
embedding_model_dict = {
    "ernie-tiny": "/root/langchain/embedding/ernie-3.0-nano-zh",
    "ernie-base": "nghuyong/ernie-3.0-base-zh",
    "text2vec-base": "shibing624/text2vec-base-chinese",
    "text2vec": "/root/langchain/embedding/text2vec-large-chinese",
    "m3e-small": "/root/langchain/embedding/m3e-small",
    "m3e-base": "/root/langchain/embedding/m3e-base",
}       
```
2.修改LLM模型路径，我的路径如下"/root/langchain/llm/"，你的路径可能不同，需要修改
```python
# llm_model_dict 处理了loader的一些预设行为，如加载位置，模型名称，模型处理器实例
# 在以下字典中修改属性值，以指定本地 LLM 模型存储位置
# 如将 "chatglm-6b" 的 "local_model_path" 由 None 修改为 "User/Downloads/chatglm-6b"
# 此处请写绝对路径
llm_model_dict = {
    "chatglm-6b-int4-qe": {
        "name": "chatglm-6b-int4-qe",
        "pretrained_model_name": "chatglm-6b-int4-qe",
        "local_model_path": "/root/langchain/llm/chatglm-6b-int4-qe",
        "provides": "ChatGLM"
    },
    "chatglm-6b-int4": {
        "name": "chatglm-6b-int4",
        "pretrained_model_name": "chatglm-6b-int4",
        "local_model_path": "/root/langchain/llm/chatglm-6b-int4",
        "provides": "ChatGLM"
    },
    "chatglm2-6b-int4": {
         "name": "chatglm2-6b-int4",
         "pretrained_model_name": "chatglm2-6b-int4",
         "local_model_path": "/root/langchain/llm/chatglm2-6b-int4",
         "provides": "ChatGLM"
   }
}
```

3.修改webui.py启动时默认加载的模型，同样在config/model_config.py里修改
```python
# Embedding model name
EMBEDDING_MODEL = "m3e-base"
```

```python
# LLM 名称
LLM_MODEL = "chatglm2-6b-int4"
```


#### 5.运行
```bash
python webui.py
```
如果没有报错，就可以打开浏览器，输入http://服务器ip:7860/，就可以看到webui的调试界面了

如果报错（大概率报错）
看你报的什么错咯，以下是我遇到的问题及解决方案

##### 报错：RuntimeError: Library cudart is not initialized
这个错误在webui.py启动时并不会直接出现，只是提示模型加载失败。
所以建议直接 python cli-demo.py，更容易暴露出错误信息

排查步骤：

1.执行init看torch是否正常初始化
```bash
python -c "import torch;torch.cuda.init();print(torch.cuda.is_available())"
```

2.检查cuda版本
```bash
python -c "import torch; print(torch.version.cuda)"
```


4.如果你的版本低于11.7(chatGLM要求的最低版本)，请升级cuda：

```bash
conda install cudatoolkit=11.7 -c nvidia
```

#### 其他报错
其他报错我就没遇到了，如果你遇到了，优先询问chatGPT，
次之自己按以下文档顺序查找相关信息(按顺序！)，最后加群或者在这里留言解答
* langchain-ChatGLM常见问题： https://github.com/imClumsyPanda/langchain-ChatGLM/blob/master/docs/FAQ.md
* 安装文档（网友提供，里面的模型修改部分跟最新版本有出入，注意分辨）：https://d29l201m55.feishu.cn/docx/LFREdCeOIoIdSQxpCG7cDpAJnJe
* langchain-ChatGLM的issues区：https://github.com/imClumsyPanda/langchain-ChatGLM/issues
* ChatGLM的issues区(有时候确实是模型加载的问题，可在这上面找)：https://github.com/THUDM/ChatGLM-6B/issues

## conda的使用
conda可以帮我们快速的创建/切换python环境，
方便我们在不同的项目中使用不同的python版本，不同的python库版本，避免版本冲突的问题。

conda分为anaconda 和 miniconda，anaconda 是一个包含了许多常用库的集合版本，miniconda 是精简版本（只包含conda、pip、zlib、python 以及它们所需的包）.
我这里使用的是miniconda，因为我不需要anaconda中的那些库，而且anaconda的安装包比较大，下载起来比较慢。

#### 1.安装conda
```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```
安装过程中会提示你是否将conda加入到环境变量中，选择yes即可。
#### 2.创建新环境
```bash
conda create -n chatglm python=3.8
```
#### 3.激活环境
```bash
conda activate chatglm
```
#### 4.退出环境
```bash
conda deactivate
```
#### 5.删除环境
```bash
conda remove -n chatglm --all
```
#### 6.查看已有环境
```bash
conda info --envs
```
#### 7.安装python库（示例，非必要）
```bash
conda install -n chatglm numpy
```


## zip压缩与解压缩
#### 1.压缩
```bash
zip -r chatglm.zip chatglm
```
#### 2.解压缩
```bash
unzip chatglm.zip
```

## rsync 的使用
rsync是一个远程数据同步工具，可通过LAN/WAN快速同步多台主机间的文件。rsync使用所谓的“rsync算法”来使本地和远程两个主机之间的文件达到同步，
这个算法只传送两个文件的不同部分，而不是每次都整份传送，因此速度保证。
我平时小文件传输一般直接scp即可，但是如果文件比较大，或者需要频繁传输，就需要用rsync了。scp是加密传输，所以相对安全，但效率也低。往往出现越传越慢的情况。
所以这里用了rsync，速度快，效率高，但是不加密，所以不要传输敏感文件。

#### 1.安装
```bash
sudo apt install rsync
```
#### 2.使用
```bash
 rsync -av --info=progress2  chatglm-6b-int4-qe.zip root@服务器ip:/root/langchain/llm/
```
其中，-av --info=progress2  是参数，chatglm-6b-int4-qe.zip 是要传输的文件，root@服务器ip:/root/langchain/llm/ 是目标地址。
-av --info=progress2参数表示：-a是归档模式，-v是显示详细信息，--info=progress2是显示传输进度。


## 扩展阅读
* [langchain-ChatGLM项目地址](https://github.com/imClumsyPanda/langchain-ChatGLM)
* [ChatGLM-6b项目地址](https://github.com/THUDM/ChatGLM-6B)
* [langchain框架文档](https://python.langchain.com/)
* [ModelWhale 平台实现](https://www.heywhale.com/mw/project/643977aa446c45f4592a1e59)
* [cuda与显卡驱动的适配问题](https://cloud.tencent.com/developer/article/2063866)
* [另一个基于langchat-ChatGLM的ui项目](https://github.com/thomas-yanxin/LangChain-ChatGLM-Webui/blob/master/docs/deploy.md)

## 声明 
`本文60%的内容由github-copilot辅助提示生成，本人部署过程中50%的问题由chatGPT解答并解决。`

<img src="https://s1.ax1x.com/2023/06/24/pCtwaJH.png" width="100%">


