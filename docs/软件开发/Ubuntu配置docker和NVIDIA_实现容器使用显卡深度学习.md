---
## tags: [ "深度学习","Ubuntu","docker","NVIDIA" ]
---

# Ubuntu配置docker和NVIDIA_实现容器使用显卡深度学习
>
> Ubuntu 20.04.6 LTS

- 安装英伟达驱动
  ![](/images/img_3.png)

- 安装docker

```bash
# 安装前准备
sudo apt update
sudo apt install vim git g++ ssh curl -y
sudo apt remove docker docker-engine docker.io containerd runc -y
sudo apt install \
apt-transport-https \
ca-certificates \
curl \
gnupg \
lsb-release -y
# 添加GPG密钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
# 安装
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io -y
# 添加配置
sudo mkdir -p /etc/docker
sudo mkdir -p /home/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": ["https://gjuibr2v.mirror.aliyuncs.com"]
}
EOF
# 安装NVIDIA容器包，支持GPU训练
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update
sudo apt install -y nvidia-container-toolkit
# 启动docker
sudo systemctl restart docker
sudo systemctl daemon-reload
sudo systemctl enable --now docker
# 测试docker是否安装成功
docker run hello-world # 成功则输出"hello-world"
```

- 安装cuda

```bash
wget https://developer.download.nvidia.com/compute/cuda/11.8.0/local_installers/cuda_11.8.0_520.61.05_linux.run
sudo sh cuda_11.8.0_520.61.05_linux.run
```

![](/images/img_4.png)

- 安装cudnn

```bash
tar -xvf cudnn-linux-x86_64-8.9.4.25_cuda11-archive.tar.xz
sudo cp cudnn-linux-x86_64-8.9.4.25_cuda11-archive/include/cudnn* /usr/local/cuda/include/
sudo cp cudnn-linux-x86_64-8.9.4.25_cuda11-archive/lib/libcudnn* /usr/local/cuda/lib64/
sudo chmod a+r /usr/local/cuda/include/cudnn*
sudo chmod a+r /usr/local/cuda/lib64/libcudnn*
# 添加环境变量
echo '# cuda env' >> .bashrc
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> .bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> .bashrc
source .bashrc
# 验证
nvcc -V
```

链接：<https://pan.baidu.com/s/1fV51Kho4hy6S9irSO7YKUw>  
提取码：zt3v  
-来自百度网盘超级会员V5的分享

- 运行深度学习容器

```bash
docker pull paddlepaddle/paddle:2.3.2-gpu-cuda11.2-cudnn8
docker run --gpus all --name dl_test -it -v /tmp/paddle:/paddle paddlepaddle/paddle:2.3.2-gpu-cuda11.2-cudnn8
/bin/bash
# 新版本
docker run --gpus all --name paddlex_test -it -v /tmp/ppdlex_test:/paddlex_test registry.baidubce.com/paddlex/paddlex:
3.0.0b0-gpu-cuda11.8-cudnn8.9-trt8.5 /bin/bash
pip install paddlex==2.1.0 -i https://mirror.baidu.com/pypi/simple
vim clas_demo.py
```

```python

  import paddlex as pdx
  from paddlex import transforms as T

  # 下载和解压蔬菜分类数据集
  veg_dataset = 'https://bj.bcebos.com/paddlex/datasets/vegetables_cls.tar.gz'
  pdx.utils.download_and_decompress(veg_dataset, path='./')

  # 定义训练和验证时的transforms
  # API说明：https://github.com/PaddlePaddle/PaddleX/blob/develop/docs/apis/transforms/transforms.md
  train_transforms = T.Compose(
  [T.RandomCrop(crop_size=224), T.RandomHorizontalFlip(), T.Normalize()])

  eval_transforms = T.Compose([
  T.ResizeByShort(short_size=256), T.CenterCrop(crop_size=224), T.Normalize()
  ])

  # 定义训练和验证所用的数据集
  # API说明：https://github.com/PaddlePaddle/PaddleX/blob/develop/docs/apis/datasets.md
  train_dataset = pdx.datasets.ImageNet(
  data_dir='vegetables_cls',
  file_list='vegetables_cls/train_list.txt',
  label_list='vegetables_cls/labels.txt',
  transforms=train_transforms,
  shuffle=True)

  eval_dataset = pdx.datasets.ImageNet(
  data_dir='vegetables_cls',
  file_list='vegetables_cls/val_list.txt',
  label_list='vegetables_cls/labels.txt',
  transforms=eval_transforms)

  # 初始化模型，并进行训练
  # 可使用VisualDL查看训练指标，参考https://github.com/PaddlePaddle/PaddleX/blob/develop/docs/visualdl.md
  num_classes = len(train_dataset.labels)
  model = pdx.cls.ResNet50_vd_ssld(num_classes=num_classes)

  # API说明：https://github.com/PaddlePaddle/PaddleX/blob/develop/docs/apis/models/classification.md
  # 各参数介绍与调整说明：https://github.com/PaddlePaddle/PaddleX/tree/develop/docs/parameters.md
  model.train(
  num_epochs=10,
  train_dataset=train_dataset,
  train_batch_size=32,
  eval_dataset=eval_dataset,
  lr_decay_epochs=[4, 6, 8],
  learning_rate=0.025,
  save_dir='output/resnet50',
  use_vdl=True)

  python clas_demo.py

![](https://blog.qianfuxin.com/wp-
content/uploads/2024/05/image-1-1024x335.png)

```
