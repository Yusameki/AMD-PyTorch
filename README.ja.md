<div align="center">

# 🍡 AMD-PyTorch

**Ubuntu 18.04 で ROCm を入れて、Docker 経由で AMD GPU 上の PyTorch を動かすまでの小記録**

<sub>[English](README.md) · **日本語** · [中文](README.zh-CN.md)</sub>

<sub>当時のノートを整理したもので、バージョン番号はもう古いです</sub>
<sub>いま実際にやるなら、[ROCm 公式ドキュメント](https://rocm.docs.amd.com/) の最新手順を見てください (´-ω-`)</sub>

</div>

---

## 目次

- [0. まず Ubuntu を更新](#0-まず-ubuntu-を更新)
- [1. ROCm の設定](#1-rocm-の設定)
- [2. Docker で ROCm 版 PyTorch を入れる](#2-docker-で-rocm-版-pytorch-を入れる)
- [3. 性能テスト](#3-性能テスト)

---

## 0. まず Ubuntu を更新

まず環境をきれいにしてから始めます。そうしないと妙なところで詰まりがちです。

```shell
sudo apt update
sudo apt upgrade -y
sudo apt install libnuma-dev -y
sudo apt autoremove -y
sudo reboot
```

## 1. ROCm の設定

### 1.1 ROCm の apt リポジトリを追加

```shell
wget -q -O - https://repo.radeon.com/rocm/rocm.gpg.key | sudo apt-key add -
echo 'deb [arch=amd64] https://repo.radeon.com/rocm/apt/debian/ xenial main' | sudo tee /etc/apt/sources.list.d/rocm.list
```

### 1.2 rocm-dkms をインストール

```shell
sudo apt update
sudo apt install rocm-dkms
sudo reboot
```

### 1.3 ユーザーに GPU へのアクセス権を付与

```shell
echo 'ADD_EXTRA_GROUPS=1' | sudo tee -a /etc/adduser.conf
echo 'EXTRA_GROUPS=video' | sudo tee -a /etc/adduser.conf
```

### 1.4 ROCm を PATH に追加

```shell
echo 'export PATH=$PATH:/opt/rocm/bin:/opt/rocm/profiler/bin:/opt/rocm/opencl/bin/x86_64' | sudo tee -a /etc/profile.d/rocm.sh
```

ここまででインストール完了です。`sudo reboot` で再起動してください。

下の 2 コマンドで確認します。**両方とも GPU 情報が出れば成功**です：

```shell
/opt/rocm/bin/rocminfo
/opt/rocm/opencl/bin/clinfo
```

> 🍮 おまけ：AMD には `nvidia-smi` みたいなものがないので、GPU の状態を見たいときは `radeontop` を試してみてください。
> （ただバージョンが古いので、動くかどうかは運次第……）
>
> ```shell
> sudo apt install radeontop
> sudo radeontop
> ```

## 2. Docker で ROCm 版 PyTorch を入れる

### 2.1 Docker をインストール

```shell
sudo apt install docker.io
```

`sudo` を減らすために、今のユーザーを docker グループに入れます：

```shell
sudo gpasswd -a $(whoami) docker
sudo chgrp docker /var/run/docker.sock
sudo reboot
```

### 2.2 公式の ROCm 版 PyTorch イメージを取得

当時使っていた ROCm は 3.9 だったので、このイメージを選びました：

```shell
docker pull rocm/pytorch:rocm3.9_ubuntu18.04_py3.6_pytorch
```

### 2.3 コンテナを起動して試す

```shell
sudo docker run -it -v $HOME:/data --privileged --rm \
  --device=/dev/kfd --device=/dev/dri --group-add video \
  rocm/pytorch:rocm3.9_ubuntu18.04_py3.6_pytorch
```

こうすると、ホスト側のホームディレクトリがコンテナ内の `/data` に同期されます。

### 2.4 PyTorch 付属のテストを走らせる

```shell
PYTORCH_TEST_WITH_ROCM=1 python test/run_test.py --verbose
```

結果：大半は通りますが、一部は落ちます。

どうやらすべての CUDA 関数に対応実装があるわけではないようです。特に `torch.backends.cudnn.benchmark = True` は有効にしないほうがいいです。

### 2.5 ついでに Docker-Compose も入れる

```shell
sudo curl -L https://github.com/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## 3. 性能テスト

Coming soon... （そのまま今まで soon のままです…(￣▽￣)"）

---

<div align="center">
<sub>このノートが役に立ったら、⭐ をぽちっとしていただけると嬉しいです ( ¯꒳¯ )</sub>
</div>
