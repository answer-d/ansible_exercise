# 1-5. 変数を使ったサーバごとの処理

## 問題

作成しないといけないディレクトリはnec_workだけではありませんでした。~~早く言え…~~  
Playbookにタスクを追加し、以下の仕様のディレクトリを作成する処理を実装して下さい。  

|item|parameter|
|:--|:--|
|パス|Target Lin#1 の場合： `/nec_work/hoge` <br>Target Lin#2 の場合： `/nec_work/poyo`|
|パーミッション|`770`|
|オーナー|`ansible`|
|所属グループ|`ansible`|

なんとも謎のフォルダ名ですが、どうやら目的はアプリケーション関連のファイル配置のようです。  
※ hogeやpoyoはAPの開発コードネームとでも思って下さい  

## 実行結果例

```console
$ ansible-playbook -i inventory setup.yml -v
Using /etc/ansible/ansible.cfg as config file

PLAY [linuxグループセットアップ] ***************************************************************************

TASK [Gathering Facts] ***************************************************************************
ok: [target_lin1]
ok: [target_lin2]

TASK [実行開始通知] ************************************************************************************
ok: [target_lin1] => {
    "msg": "192.168.250.237に対してPlaybookの実行を開始します"
}
ok: [target_lin2] => {
    "msg": "192.168.159.104に対してPlaybookの実行を開始します"
}

TASK [workディレクトリ作成] ******************************************************************************
ok: [target_lin1] => {"changed": false, "gid": 0, "group": "root", "mode": "0777", "owner": "root", "path": "/nec_work", "secontext": "unconfined_u:object_r:default_t:s0", "size": 6, "state": "directory", "uid": 0}
ok: [target_lin2] => {"changed": false, "gid": 0, "group": "root", "mode": "0777", "owner": "root", "path": "/nec_work", "secontext": "unconfined_u:object_r:default_t:s0", "size": 18, "state": "directory", "uid": 0}

TASK [AP用ディレクトリ作成] *******************************************************************************
changed: [target_lin1] => {"changed": true, "gid": 2001, "group": "ansible", "mode": "0770", "owner": "ansible", "path": "/nec_work/hoge", "secontext": "unconfined_u:object_r:default_t:s0", "size": 6, "state": "directory", "uid": 2001}
ok: [target_lin2] => {"changed": false, "gid": 2001, "group": "ansible", "mode": "0770", "owner": "ansible", "path": "/nec_work/poyo", "secontext": "unconfined_u:object_r:default_t:s0", "size": 6, "state": "directory", "uid": 2001}

PLAY RECAP ***************************************************************************************
target_lin1                : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
target_lin2                : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## ヒント

<details>
    <summary>clip to expand</summary>

- ディレクトリの名前を変数化し、実行対象ノードごとに異なる値を設定することで目的の処理を実現することができます
- 実行対象ノードごとに異なる変数を定義するには、host_varsを用いるのが楽でオススメです
    - 他にも、インベントリファイル内でホスト変数を定義する方法などもあります  
      どちらでも正解ですので、このあたりを参考に実装してみて下さい  
      <https://www.atmarkit.co.jp/ait/articles/1610/05/news013.html>  
      → `ホスト／グループごとに変数を定義する`

</details>

## 解答

<details>
    <summary>clip to expand</summary>

### コード

#### ファイル構成

```plain
│  setup.yml
│
└─host_vars
        target_lin1.yml
        target_lin2.yml
```

#### setup.yml

```yaml
---
- name: linuxグループセットアップ
  hosts: linux
  tasks:
    - name: 実行開始通知
      debug:
        msg: "{{ ansible_host }}に対してPlaybookの実行を開始します"

    - name: workディレクトリ作成
      become: true
      file:
        path: /nec_work
        mode: "777"
        state: directory

    - name: AP用ディレクトリ作成
      file:
        path: /nec_work/{{ app_dir_name }}
        mode: "770"
        state: directory
```

#### host_vars/target_lin1.yml

```yaml
---
app_dir_name: hoge
```

#### host_vars/target_lin2.yml

```yaml
---
app_dir_name: poyo
```

[回答例の実ファイルはこちら](./answer/)  

### 解説

- インベントリで変数を提供するための推奨される方法は、 `host_vars` と `group_vars` という2つのディレクトリにあるファイルにそれらを定義することです
    - たとえば、グループ `servers` の変数を定義するためには、YAMLファイル(あるいはディレクトリ) `group_vars/servers` を作成します
    - また、特定ホスト `node1` 専用の変数を定義するためには、YAMLファイル(あるいはディレクトリ) `host_vars/node1` を作成します
- 今回のディレクトリ作成はansibleユーザの権限範囲にて実施できるため、 `become` ディレクティブは無くてもOKです
    - もちろん入れてあっても問題ありません。その場合はfileモジュールのパラメータ `owner` や `group` でディレクトリの属性を指定してあげましょう

</details>

## Navigation

前：[1-4. 具体的な処理を実行してみよう](../1-4_essential-playbook/README.md)  
次：[1-6. ファイルコピー(copyモジュール)](../1-6_copy-module/README.md)  

[Top](../README.md)  
