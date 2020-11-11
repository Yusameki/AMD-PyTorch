# AMD-PyTorch

Ubuntu18.04用

・ROCmを配置します

・dockerをインストールします

・pytorchを動かします



## Ubuntuの更新

以下のコマンドを使って最新の環境を確保します．

``` shell
sudo apt update
sudo apt upgrade -y
sudo apt install libnuma-dev -y
sudo apt autoremove -y
sudo reboot
```



## ROCmの配置

1.ROCmのaptレポジトリを追加します．

``` shell
wget -q -O - https://repo.radeon.com/rocm/rocm.gpg.key | sudo apt-key add -
echo 'deb [arch=amd64] https://repo.radeon.com/rocm/apt/debian/ xenial main' | sudo tee /etc/apt/sources.list.d/rocm.list
```



2.ROCmのrocm-dkmsパッケージをインストールします．

``` shell
sudo apt update
sudo apt install rocm-dkms
sudo reboot
```



3.ユーザーにGPU訪問権を与えます．

```shell
echo 'ADD_EXTRA_GROUPS=1' | sudo tee -a /etc/adduser.conf
echo 'EXTRA_GROUPS=video' | sudo tee -a /etc/adduser.conf
```



4.ROCmをPATHに追加します．

```shell
echo 'export PATH=$PATH:/opt/rocm/bin:/opt/rocm/profiler/bin:/opt/rocm/opencl/bin/x86_64' | sudo tee -a /etc/profile.d/rocm.sh
```

これにてインストールが完了しました．

``sudo reboot``で再起動します．



以下のコマンドでインストールを検証できます．

```shell
/opt/rocm/bin/rocminfo
/opt/rocm/opencl/bin/clinfo
```

両方ともGPUの情報がリストされたらインストール成功となります．



> 任意

``nvidia-smi``的な機能が存在しないため，radeontopというソフトでGPUを監視できるかもしれません．

(バージョンが古いため，何とも言えないが)

```shell
sudo apt install radeontop
sudo radeontop
```



## Dockerを用いてROCm-PyTorchをインストール

1.Dockerをインストールします．

```shell
sudo apt install docker.io
```

便利上，Dockerをsudoなしでも使えるようにします．

```shell
sudo gpasswd -a $(whoami) docker
sudo chgrp docker /var/run/docker.sock
sudo reboot
```



2.Docker Hubより，オフィシャルのROCm対応PyTorchイメージを取得します．

   今回使用するROCmのバージョンは3.9であるため，以下のイメージを使用します．

```shell
docker pull rocm/pytorch:rocm3.9_ubuntu18.04_py3.6_pytorch
```



3.テスト目的で，以下のコマンドでコンテナを起動します．

```shell
sudo docker run -it -v $HOME:/data --privileged --rm --device=/dev/kfd --device=/dev/dri --group-add video rocm/pytorch:rocm3.9_ubuntu18.04_py3.6_pytorch
```

ユーザーのhomeフォルダがコンテナ内の/dataフォルダと同期した状態で起動しました．



4.PyTorchのテストを行います．

```shell
PYTORCH_TEST_WITH_ROCM=1 python test/run_test.py --verbose
```

結果的に，ほとんどのテストにクリアできるが，一部で失敗することもあります．

CUDA関数が全て対応しているわけではない感じでした，特に

``torch.backends.cudnn.benchmark = True``は使わない方が良さそうです．

5.Docker-Composeをインストールします

```shell
sudo curl -L https://github.com/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## Performance test

Coming soon...
