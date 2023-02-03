# Readme.md
## ログイン
ログインは、コマンドプロンプトで以下のコマンドを打つ。
```sh
ssh msuzuki@10.1.1.1
```
基本はssh <username>@<ip address>の形。

毎度打つのが面倒であれば、`U:\users\a12345\.ssh\config`に定義しておくと便利。

```
Host example
    User msuzuki
    HostName 10.1.1.1
```
この例では`ssh example`でログインできるようになる。


## 初期設定
Proxyでライブラリを取得できないことがあるため、`apt`と`wget`で外部に接続できることが望ましい。<br>
また、`sudo`(管理者)権限がないと初期設定はできない。

```sh
sudo apt -y install git curl vim tmux 
sudo apt -y install ant nkf r-base-core zlib1g libssl-dev libbz2-dev python-is-python3 python-dev-is-python3 zlib1g-dev libsqlite3-dev libffi-dev libmysqlclient-dev liblzma-dev maven
```

### GPUマシンでPyTorch等でCUDAを利用する場合
NVIDIAのGPUがないマシンではこの項目は不要。<br>

#### CUDAのインストール
下記の例では、Ubuntu20.04のCUDAをインストールする。<br>
最新のCUDAとOS distributionは[developer.nvidia.com/cuda-downloads](https://developer.nvidia.com/cuda-downloads)から確認する。<br>
参考までに、2023年2月3日時点でPyTorchはCUDA 12に対応しておらず、CUDA 11はUbuntu 20.04用までしかないためこちらをインストールする。<br>
`<proxy address>`には、各環境のProxyのURLを使用する。

```sh
sudo apt-key adv --keyserver-option http-proxy=<proxy address> --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/3bf863cc.pub
sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/3bf863cc.pub
sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/ /"
sudo apt update
sudo apt install -y cuda cuda-drivers
```

#### cuDNNのインストール
インストールに必要なパッケージは、ログインしないと作成できなくなったため、Azureではなく手元のブラウザで一旦ダウンロードする。<br>
基本的には上の最新版からダウンロードすればよい

- [cuDNN Download](https://developer.nvidia.com/rdp/cudnn-download)
- [cuDNN Archive](https://developer.nvidia.com/rdp/cudnn-archive)

ダウンロード出来たら、`scp`コマンドを用いて転送する。<br>
ファイル名は環境で異なるので注意。<br>
先ほどの`ssh`と似ており、`scp <file name> <username>@<ipaddress>`の形で使う。

```sh
scp cudnn-local-repo-ubuntu2204-8.7.0.84_1.0-1_amd64.deb msuzuki@10.1.1.1
```

転送が完了したら、再度`ssh msuzuki@10.1.1.1`のようにログインしなおす。<br>
以下のコマンドを実行してcuDNNをインストールする。

```sh
sudo dpkg -i cudnn-local-repo-ubuntu2204-8.7.0.84_1.0-1_amd64.deb
sudo apt update
sudo apt install libcudnn8-dev
```

### Python
1つのバージョンで仮想環境を用いない場合は上記(`sudo apt install -y python-is-python3 python-dev-is-python3`)でインストールしたPythonを使用してもよい。<br>
一方で、複数のユーザーやディレクトリでライブラリを使い分けたい場合は[Pyenv](https://github.com/pyenv/pyenv)や[Poetry](https://github.com/python-poetry/poetry)を使う。<br>
Anacondaも使えるが、依存環境解決やインストール速度に難があるため、近年はPyenvとPoetryの併用が多い印象。

### Pyenv 
詳細な使い方は[リポジトリ](https://github.com/pyenv/pyenv)を参照のこと。

```sh
git config --global http.proxy <proxy address>
echo 'git clone https://github.com/pyenv/pyenv.git ~/.pyenv'
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(pyenv init --path)"' >> ~/.bash_profile
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bash_profile
echo 'test -r ~/.bashrc && . ~/.bashrc' >> ~/.bash_profile
source ~/.bash_profile
# python 3.10.9をインストールする場合
https_proxy=<proxy address> pyenv install 3.10.9
pyenv global 3.10.9
```


## Poetry
詳細な使い方は[リポジトリ](https://github.com/python-poetry/poetry)を参照のこと。

```sh
curl -sSL https://install.python-poetry.org -x http:/webprx1.nikkoam.com:80 | https_proxy=http://webprx1.nikkoam.com:80 python -
poetry config virtualenvs.in-project true
```

## データ転送
初期設定で用いた`scp`コマンドを使うのが追加インストールも不要で簡単である。<br>
`-r`オプションをつければディレクトリの転送も可能。<br>
GUI上で転送したい場合は、WinSCPやVSCodeを使える。<br>
ただし、VSCodeでの転送は速度に難がある印象。
