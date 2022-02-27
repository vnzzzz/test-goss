# test-goss

## 目的

- goss を試す
- goss バイナリは ansible で配る
- テスト対象の VM は vagrant + virtualbox で作成する

## 環境

- mac

## 手順

### 諸々のインストール

- goss を github から取得する

  ```bash
  curl -L https://github.com/aelsabbahy/goss/releases/download/v0.3.9/goss-linux-amd64 -o goss
  ```

  バイナリ形式ですぐに実行可能なファイルとして取得される。
  ansible で配るため、のちの工程で適当なディレクトリに移動させる。

- ansible のインストール

  ```terminal
  brew install ansible
  ```

- virtual box

  ```terminal
  brew install --cask virtualbox
  ```

- vagrant

  ```terminal
  brew install --cask vagrant
  ```

### vagrant を利用し、virtual box 上に centos7 の VM を作成する

- vagrant box を追加する

  ```bash
  vagrant box add centos/7
  ```

  provider を何にするか聞かれるので、`3) virtualbox`を選択する

  インストールした box を確認するときは

  ```bash
  vagrant box list
  ```

- ディレクトリ作成&移動する

  仮想マシン作成用のディレクトリ`test-centos7`を作成し、Vagrantfile を作成

  ```bash
  cd vagrant
  mkdir test-centos7
  cd test-centos7
  ```

- Vagrantfile を作成する

  ```bash
  vagrant init centos/7
  ```

  `config.vm.network "private_network"`の部分の設定は、バッティングしないよう適切な IP アドレスを設定する

- VM を起動する

  ```bash
  vagrant up
  ```

- VM の状態を確認する

  ```bash
  vagrant status
  ```

- ssh で VM に接続する

  ```bash
  vagrant ssh
  ```
