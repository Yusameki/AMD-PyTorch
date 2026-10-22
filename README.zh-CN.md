<div align="center">

# 🍡 AMD-PyTorch

**Ubuntu 18.04 上把 ROCm 装好、用 Docker 让 PyTorch 跑在 AMD GPU 上的小记录**

<sub>[English](README.md) · [日本語](README.ja.md) · **中文**</sub>

<sub>这是当年那份笔记整理后的版本，里面的版本号都还是旧的了</sub>
<sub>真要现在动手的话，记得对照 [ROCm 官方文档](https://rocm.docs.amd.com/) 的最新步骤 (´-ω-`)</sub>

</div>

---

## 目录

- [0. 先更新 Ubuntu](#0-先更新-ubuntu)
- [1. 配置 ROCm](#1-配置-rocm)
- [2. 用 Docker 装 ROCm 版 PyTorch](#2-用-docker-装-rocm-版-pytorch)
- [3. 性能测试](#3-性能测试)

---

## 0. 先更新 Ubuntu

先把环境擦干净再开始，不然很容易在奇怪的地方卡住。

```shell
sudo apt update
sudo apt upgrade -y
sudo apt install libnuma-dev -y
sudo apt autoremove -y
sudo reboot
```

## 1. 配置 ROCm

### 1.1 添加 ROCm 的 apt 源

```shell
wget -q -O - https://repo.radeon.com/rocm/rocm.gpg.key | sudo apt-key add -
echo 'deb [arch=amd64] https://repo.radeon.com/rocm/apt/debian/ xenial main' | sudo tee /etc/apt/sources.list.d/rocm.list
```

### 1.2 安装 rocm-dkms

```shell
sudo apt update
sudo apt install rocm-dkms
sudo reboot
```

### 1.3 给用户 GPU 访问权限

```shell
echo 'ADD_EXTRA_GROUPS=1' | sudo tee -a /etc/adduser.conf
echo 'EXTRA_GROUPS=video' | sudo tee -a /etc/adduser.conf
```

### 1.4 把 ROCm 加进 PATH

```shell
echo 'export PATH=$PATH:/opt/rocm/bin:/opt/rocm/profiler/bin:/opt/rocm/opencl/bin/x86_64' | sudo tee -a /etc/profile.d/rocm.sh
```

到这里就装完了，`sudo reboot` 重启一下。

用下面两条命令验证一下，**两个都能列出 GPU 信息**就说明成功了：

```shell
/opt/rocm/bin/rocminfo
/opt/rocm/opencl/bin/clinfo
```

> 🍮 可选：AMD 这边没有 `nvidia-smi` 那种东西，想看显卡状态可以试试 `radeontop`。
> （不过版本有点老，能不能用要看运气……）
>
> ```shell
> sudo apt install radeontop
> sudo radeontop
> ```

## 2. 用 Docker 装 ROCm 版 PyTorch

### 2.1 装 Docker

```shell
sudo apt install docker.io
```

为了少打一点 `sudo`，把当前用户塞进 docker 组：

```shell
sudo gpasswd -a $(whoami) docker
sudo chgrp docker /var/run/docker.sock
sudo reboot
```

### 2.2 拉官方 ROCm 版 PyTorch 镜像

当时用的 ROCm 版本是 3.9，所以镜像选这个：

```shell
docker pull rocm/pytorch:rocm3.9_ubuntu18.04_py3.6_pytorch
```

### 2.3 起容器试一下

```shell
sudo docker run -it -v $HOME:/data --privileged --rm \
  --device=/dev/kfd --device=/dev/dri --group-add video \
  rocm/pytorch:rocm3.9_ubuntu18.04_py3.6_pytorch
```

这样启动后，宿主机的 home 目录会同步到容器里的 `/data`。

### 2.4 跑 PyTorch 自带的测试

```shell
PYTORCH_TEST_WITH_ROCM=1 python test/run_test.py --verbose
```

结果：大部分测试能过，少数会挂。

感觉并不是所有 CUDA 函数都有对应实现，尤其是 `torch.backends.cudnn.benchmark = True` 最好别开。

### 2.5 顺便装上 Docker-Compose

```shell
sudo curl -L https://github.com/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## 3. 性能测试

Coming soon... （然后就一直 soon 到了现在…(￣▽￣)"）

---

<div align="center">
<sub>如果这份笔记也帮到了你，按个 ⭐ 就很好啦 ( ¯꒳¯ )</sub>
</div>
