---
layout: post
title:  保姆级教程：如何本地部署私有chatglm模型及知识库扩展调试
date: 2023-06-24 14:23:20
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
* 系统：Ubuntu 18.04
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
git clone
```
#### 3.安装依赖
需要提前安装飞浆的依赖环境libs

然后安装python依赖：
```bash
pip install -r requirements.txt
```
检查有无问题



#### 4.下载模型
如果你网络环境好，人在美国刚下飞机，可以不用手动下载，直接运行python webui.py，会自动下载模型
如果不好（大概率不好），请手动下载模型，然后解压到chatglm目录下

```bash
cd chatglm
wget https://paddlenlp.bj.bcebos.com/models/transformers/chatglm/chatglm_model.tar.gz
tar -zxvf chatglm_model.tar.gz
```

#### 5.运行
```bash
python webui.py
```
如果没有报错，就可以打开浏览器，输入http://localhost:5000/，就可以看到chatglm的webui界面了


如果报错（大概率报错）
看你报的什么错咯，以下是我遇到的问题及解决方案

##### 1.报错：ModuleNotFoundError: No module named 'paddlehub'
解决方案：
```bash
pip install paddlehub
```
