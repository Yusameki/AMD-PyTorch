<div align="center">

# 🍡 AMD-PyTorch

**Running PyTorch on an AMD GPU — ROCm on Ubuntu 18.04, plus Docker**

<sub>**English** · [中文](README.zh-CN.md)</sub>

<sub>Old notes, tidied up — the version numbers in here are all outdated</sub>
<sub>If you're doing this today, follow the current steps in the [official ROCm docs](https://rocm.docs.amd.com/) (´-ω-`)</sub>

</div>

---

## Contents

- [0. Update Ubuntu first](#0-update-ubuntu-first)
- [1. Set up ROCm](#1-set-up-rocm)
- [2. Install ROCm PyTorch via Docker](#2-install-rocm-pytorch-via-docker)
- [3. Benchmarks](#3-benchmarks)

---

## 0. Update Ubuntu first

Wipe the environment clean before you start — otherwise you'll get stuck somewhere strange.

```shell
sudo apt update
sudo apt upgrade -y
sudo apt install libnuma-dev -y
sudo apt autoremove -y
sudo reboot
```

## 1. Set up ROCm

### 1.1 Add the ROCm apt repository

```shell
wget -q -O - https://repo.radeon.com/rocm/rocm.gpg.key | sudo apt-key add -
echo 'deb [arch=amd64] https://repo.radeon.com/rocm/apt/debian/ xenial main' | sudo tee /etc/apt/sources.list.d/rocm.list
```

### 1.2 Install rocm-dkms

```shell
sudo apt update
sudo apt install rocm-dkms
sudo reboot
```

### 1.3 Give your user GPU access

```shell
echo 'ADD_EXTRA_GROUPS=1' | sudo tee -a /etc/adduser.conf
echo 'EXTRA_GROUPS=video' | sudo tee -a /etc/adduser.conf
```

### 1.4 Add ROCm to PATH

```shell
echo 'export PATH=$PATH:/opt/rocm/bin:/opt/rocm/profiler/bin:/opt/rocm/opencl/bin/x86_64' | sudo tee -a /etc/profile.d/rocm.sh
```

That's the install done — `sudo reboot` to apply.

Verify with these two commands. If **both list your GPU**, you're good:

```shell
/opt/rocm/bin/rocminfo
/opt/rocm/opencl/bin/clinfo
```

> 🍮 Optional: AMD has no equivalent of `nvidia-smi`. For GPU status, try `radeontop`.
> (It's a bit old though — whether it works is down to luck…)
>
> ```shell
> sudo apt install radeontop
> sudo radeontop
> ```

## 2. Install ROCm PyTorch via Docker

### 2.1 Install Docker

```shell
sudo apt install docker.io
```

To save typing `sudo` all the time, add your user to the docker group:

```shell
sudo gpasswd -a $(whoami) docker
sudo chgrp docker /var/run/docker.sock
sudo reboot
```

### 2.2 Pull the official ROCm PyTorch image

The ROCm version at the time was 3.9, so this is the image:

```shell
docker pull rocm/pytorch:rocm3.9_ubuntu18.04_py3.6_pytorch
```

### 2.3 Start a container and try it

```shell
sudo docker run -it -v $HOME:/data --privileged --rm \
  --device=/dev/kfd --device=/dev/dri --group-add video \
  rocm/pytorch:rocm3.9_ubuntu18.04_py3.6_pytorch
```

With this, your host home directory is mounted at `/data` inside the container.

### 2.4 Run PyTorch's own test suite

```shell
PYTORCH_TEST_WITH_ROCM=1 python test/run_test.py --verbose
```

Result: most tests pass, a few fail.

Not every CUDA function has a ROCm counterpart, it seems — in particular, better leave `torch.backends.cudnn.benchmark = True` off.

### 2.5 Also grab Docker Compose

```shell
sudo curl -L https://github.com/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## 3. Benchmarks

Coming soon... (and it has been "soon" ever since… (￣▽￣)")

---

<div align="center">
<sub>If these notes helped you too, a ⭐ would be lovely ( ¯꒳¯ )</sub>
</div>
