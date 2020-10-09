# 1-8. ロールを使ってみよう

## 問題

ここまでで作成してきたように、Playbookを1つのファイルで管理することは技術的には可能です。  
しかしそのうち、作成したPlaybookを分割して再利用可能にしたり、汎用化したりしたいと思うようになります。  
Ansibleではこの実現のため、ロールというものを用意しています。  

ロールは基本的にはタスクのリストを小さい単位に分割する機能であり、そこに以下のような便利な仕組みがいくつか付与されている感じのモノです。  

- そのロールで使いたい変数を定義する機能
- メタ情報の付与(ロール間の依存関係など)
- ハンドラ(特定のタスクを実行したときにのみ実行されるタスクの定義方法)の自動読み込み
- `copy` モジュールや `template` モジュールでコピーする元ファイルの格納場所を提供

※ 詳細 → <https://docs.ansible.com/ansible/2.9_ja/user_guide/playbooks_reuse_roles.html>

さて、この「ロール」を利用して、これまでに作成してきた以下の3タスクを1つのロール(名前: `common_setting` )にまとめてみて下さい。  

- workディレクトリ作成
- AP用ディレクトリ作成
- hosts配布

## 実行結果例

```console
PLAY [linuxグループセットアップ] ************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************
ok: [target_lin2]
ok: [target_lin1]

TASK [実行開始通知] *********************************************************************************************************
ok: [target_lin1] => {
    "msg": "172.30.0.2に対してPlaybookの実行を開始します"
}
ok: [target_lin2] => {
    "msg": "172.30.0.3に対してPlaybookの実行を開始します"
}

TASK [共通設定] ***********************************************************************************************************

TASK [common_setting : workディレクトリ作成] **********************************************************************************
ok: [target_lin1] => {"changed": false, "gid": 0, "group": "root", "mode": "0777", "owner": "root", "path": "/nec_work", "size": 4096, "state": "directory", "uid": 0}
ok: [target_lin2] => {"changed": false, "gid": 0, "group": "root", "mode": "0777", "owner": "root", "path": "/nec_work", "size": 4096, "state": "directory", "uid": 0}

TASK [common_setting : AP用ディレクトリ作成] ***********************************************************************************
ok: [target_lin1] => {"changed": false, "gid": 1000, "group": "ansible", "mode": "0770", "owner": "ansible", "path": "/nec_work/hoge", "size": 4096, "state": "directory", "uid": 1000}
ok: [target_lin2] => {"changed": false, "gid": 1000, "group": "ansible", "mode": "0770", "owner": "ansible", "path": "/nec_work/poyo", "size": 4096, "state": "directory", "uid": 1000}

TASK [common_setting : hosts配布] ***************************************************************************************
ok: [target_lin2] => {"changed": false, "checksum": "823980a35777b84d563c534123cf6a28a8ce1615", "dest": "/etc/hosts.tmp", "gid": 0, "group": "root", "mode": "0644", "owner": "root", "path": "/etc/hosts.tmp", "size": 73, "state": "file", "uid": 0}
ok: [target_lin1] => {"changed": false, "checksum": "823980a35777b84d563c534123cf6a28a8ce1615", "dest": "/etc/hosts.tmp", "gid": 0, "group": "root", "mode": "0644", "owner": "root", "path": "/etc/hosts.tmp", "size": 73, "state": "file", "uid": 0}

PLAY RECAP ************************************************************************************************************
target_lin1                : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
target_lin2                : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## ヒント

<details>
    <summary>clip to expand</summary>

- ロールを使うにはにはAnsibleにより定められたディレクトリ構造に従ってファイルを作成する必要があります、まずはそれを調べましょう
    - 上述の公式ドキュメントが参考になります
    - 今回の場合、ロール内に作成すべきフォルダは `tasks` および `templates` あたりになるでしょう
- `setup.yml` からロールの呼び出す方法いくつかありますが、今回の場合は `include_role` モジュールを用いるのがわかりやすいかと思います

</details>

## 解答

<details>
    <summary>clip to expand</summary>

### コード

#### ファイル構成

```plain
.
├── host_vars
│   ├── target_lin1.yml
│   └── target_lin2.yml
├── inventory
├── roles
│   └── common_setting
│       ├── tasks
│       │   └── main.yml
│       └── templates
│           └── hosts.j2
└── setup.yml
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

    - name: 共通設定
      include_role:
        name: common_setting

```

#### roles/common_settings/tasks/main.yml

```yaml
---
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

- name: hosts配布
  become: true
  template:
    src: hosts.j2
    dest: /etc/hosts
```

#### roles/common_settings/templates/hosts.j2

```
127.0.0.1       localhost

{% for host in groups["all"] %}
  {{- host }} {{ hostvars[host]["ansible_host"] }}
{% endfor %}
```

### host_vars/target_lin1.yml

```yaml
---
app_dir_name: hoge
```

### host_vars/target_lin2.yml

```yaml
---
app_dir_name: poyo
```

[raw file](./answer/)  

### 解説

- Ansibleにより定められた通り、 ロール名のディレクトリ配下に `tasks` ディレクトリを作成し、その中に `main.yml` という名前のファイルを用意します
    - このファイルが、ロールのインクルード時に呼び出されるファイルとなります
- また、今回は `template` モジュールによるファイルコピーを行っているので、テンプレート置き場となる `templates` ディレクトリを作成します
    - ロール内で呼び出される `template` では、 `src` 指定するパスが暗黙的に該当ロール内の `templates` ディレクトリを参照するようになります(便利機能の1つ)  
      よって、以下のような解答のような文法が成立します  
      ※ `src` に指定しているパスに、 `templates` ディレクトリを明示していないのにファイルがコピーされていることを確認して下さい

        ```yaml
        - name: hosts配布
        become: true
        template:
          src: hosts.j2
          dest: /etc/hosts
        ```

    - 同様に `copy` モジュールを使用する場合は、同ロール内の `files` ディレクトリが暗黙的な解決先となります

</details>

## Navigation

前：[1-7. ファイルコピー(templateモジュール)](1-7_template-module/README.md)  
次：なし (基礎編はこれで終わりです)

[Top](../README.md)  
