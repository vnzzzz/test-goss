# test-goss

## 目的

- goss を試す
- goss バイナリは ansible で配る
- テスト対象の VM は vagrant + virtualbox で作成する

## 環境

- centos7
- virtual box
- mac

## ディレクトリ構成

概説

- vagrant 設定は `./vagrant` 配下に用意する
- ansible 設定は `./ansible` 配下に用意する

構成

- ディレクトリ構成

  ```bash
  .
  ├── README.md
  ├── ansible
  │   ├── inventories
  │   │   └── hosts
  │   ├── roles
  │   │   ├── goss
  │   │   │   ├── files
  │   │   │   │   ├── goss
  │   │   │   │   └── goss.yaml
  │   │   │   └── tasks
  │   │   │       └── main.yml
  │   │   └── yum
  │   │       ├── files
  │   │       │   └── nginx.repo
  │   │       └── tasks
  │   │           └── main.yml
  │   └── site.yml
  └── vagrant
      └── test-centos7
          ├── .vagrant
          └── Vagrantfile
  ```

## 手順

### 諸々のインストール

- goss を github から取得する

  ```bash
  curl -L https://github.com/aelsabbahy/goss/releases/download/v0.3.9/goss-linux-amd64 -o goss
  ```

  バイナリ形式ですぐに実行可能なファイルとして取得される。
  ansible で配るため、のちの工程で`./ansible/roles/goss/files`に格納する。

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

  provider を何にするか聞かれるので、`3) virtualbox`を選択する。

  インストールした box を確認するときは

  ```bash
  vagrant box list
  ```

- ディレクトリ作成&移動する

  仮想マシン作成用のディレクトリ`test-centos7`を作成し、Vagrantfile を作成する。

  ```bash
  cd vagrant
  mkdir test-centos7
  cd test-centos7
  ```

- Vagrantfile を作成する

  ```bash
  vagrant init centos/7
  ```

  `config.vm.network "private_network"`の部分の設定は、バッティングしないよう適切な IP アドレスを設定する。

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

  素の ssh でログインするときは、下記を実行する。

  ```bash
  ssh -i vagrant/test-centos7/.vagrant/machines/default/virtualbox/private_key vagrant@192.168.56.10
  ```

## ansible を利用して goss を実行

- private_key をコピー

  ```bash
  cp vagrant/test-centos7/.vagrant/machines/default/virtualbox/private_key ansible/inventories/
  ```

- ansible playbook を実行

  ```bash
  cd ansible
  ansible-playbook -i inventories/hosts site.yml
  ```

  設定が正しければ、下記のように ansible の実行結果が表示され、その中で goss の実行結果が出力される。

  ```bash
  PLAY [target-servers] **********************************************************************************************************

  TASK [Gathering Facts] *********************************************************************************************************
  ok: [192.168.56.10]

  TASK [yum : add nginx repo] ****************************************************************************************************
  ok: [192.168.56.10]

  TASK [yum : install nginx] *****************************************************************************************************
  ok: [192.168.56.10]

  TASK [yum : restart & enable nginx] ********************************************************************************************
  changed: [192.168.56.10]

  TASK [goss : Copy goss to remote host] *****************************************************************************************
  changed: [192.168.56.10] => (item={'file': 'files/goss', 'mode': '751'})
  changed: [192.168.56.10] => (item={'file': 'files/goss.yaml', 'mode': '666'})

  TASK [goss : Exec Goss Validate] ***********************************************************************************************
  changed: [192.168.56.10]

  TASK [goss : Goss results] *****************************************************************************************************
  ok: [192.168.56.10] => {
    "msg": {
      "changed": true,
      "cmd": [
        "./goss",
        "validate",
        "--format",
        "documentation"
      ],
      "delta": "0:00:00.047702",
      "end": "2022-02-27 13:05:08.362238",
      "failed": false,
      "msg": "",
      "rc": 0,
      "start": "2022-02-27 13:05:08.314536",
      "stderr": "",
      "stderr_lines": [],
      "stdout": "Title: nginx が起動していること\nProcess: nginx: running: matches expectation: [true]\nTitle: 80 ポートをリッスンしていること\nPort: tcp:80: listening: matches expectation: [true]\nTitle: nginxの設定ファイルが存在すること\nFile: /etc/nginx/nginx.conf: exists: matches expectation: [true]\nTitle: nginx 1.20.2 がインストールされていること\nPackage: nginx: installed: matches expectation: [true]\nPackage: nginx: version: matches expectation: [[\"1.20.2\"]]\nTitle: nginx がサービスに登録されていること\nService: nginx: enabled: matches expectation: [true]\nService: nginx: running: matches expectation: [true]\nTitle: httpで疎通できること\nHTTP: http://localhost: status: matches expectation: [200]\n\n\nTotal Duration: 0.043s\nCount: 8, Failed: 0, Skipped: 0",
      "stdout_lines": [
        "Title: nginx が起動していること",
        "Process: nginx: running: matches expectation: [true]",
        "Title: 80 ポートをリッスンしていること",
        "Port: tcp:80: listening: matches expectation: [true]",
        "Title: nginxの設定ファイルが存在すること",
        "File: /etc/nginx/nginx.conf: exists: matches expectation: [true]",
        "Title: nginx 1.20.2 がインストールされていること",
        "Package: nginx: installed: matches expectation: [true]",
        "Package: nginx: version: matches expectation: [[\"1.20.2\"]]",
        "Title: nginx がサービスに登録されていること",
        "Service: nginx: enabled: matches expectation: [true]",
        "Service: nginx: running: matches expectation: [true]",
        "Title: httpで疎通できること",
        "HTTP: http://localhost: status: matches expectation: [200]",
        "",
        "",
        "Total Duration: 0.043s",
        "Count: 8, Failed: 0, Skipped: 0"
      ]
    }
  }

  TASK [goss : Delete Goss] ******************************************************************************************************
  changed: [192.168.56.10] => (item=/root/goss)
  changed: [192.168.56.10] => (item=/root/goss.yaml)

  PLAY RECAP *********************************************************************************************************************
  192.168.56.10              : ok=8    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
  ```
