# 1-7. ファイルコピー(templateモジュール)

## 問題

前問のPlaybookによりhostsを配布できるようになりましたが、ファイルの内容が静的なので少し使い勝手が悪いです。  
具体的には、以下のような点に課題があります。

- 環境(本番環境/ステージング環境/開発環境)が複数存在するシステムの場合、環境ごとにファイルを用意する必要がある
- サーバ台数やIPアドレスが変化した場合、インベントリファイルだけでなくhostsファイルも更新しなければならない

copyモジュールによって行っていたhostsファイルの配布処理の部分を修正し、  
インベントリファイルに記載されているホストのすべてに対する名前解決が可能なhostsファイルを **動的に** 作成し、  
`target_lin1` と `target_lin2` の `/etc/hosts` として配布するように実装して下さい。  

## 実行結果例

```console
PLAY [linuxグループセットアップ] *************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [target_lin2]
ok: [target_lin1]

TASK [実行開始通知] **********************************************************************************************************************
ok: [target_lin1] => {
    "msg": "172.30.0.2に対してPlaybookの実行を開始します"
}
ok: [target_lin2] => {
    "msg": "172.30.0.3に対してPlaybookの実行を開始します"
}

TASK [workディレクトリ作成] ****************************************************************************************************************
ok: [target_lin1] => {"changed": false, "gid": 0, "group": "root", "mode": "0777", "owner": "root", "path": "/nec_work", "size": 4096, "state": "directory", "uid": 0}
ok: [target_lin2] => {"changed": false, "gid": 0, "group": "root", "mode": "0777", "owner": "root", "path": "/nec_work", "size": 4096, "state": "directory", "uid": 0}

TASK [AP用ディレクトリ作成] *****************************************************************************************************************
ok: [target_lin1] => {"changed": false, "gid": 1000, "group": "ansible", "mode": "0770", "owner": "ansible", "path": "/nec_work/hoge", "size": 4096, "state": "directory", "uid": 1000}
ok: [target_lin2] => {"changed": false, "gid": 1000, "group": "ansible", "mode": "0770", "owner": "ansible", "path": "/nec_work/poyo", "size": 4096, "state": "directory", "uid": 1000}

TASK [hosts配布] *********************************************************************************************************************
ok: [target_lin1] => {"changed": false, "checksum": "823980a35777b84d563c534123cf6a28a8ce1615", "dest": "/etc/hosts", "gid": 0, "group": "root", "mode": "0644", "owner": "root", "path": "/etc/hosts", "size": 73, "state": "file", "uid": 0}
ok: [target_lin2] => {"changed": false, "checksum": "823980a35777b84d563c534123cf6a28a8ce1615", "dest": "/etc/hosts", "gid": 0, "group": "root", "mode": "0644", "owner": "root", "path": "/etc/hosts", "size": 73, "state": "file", "uid": 0}

PLAY RECAP *************************************************************************************************************************
target_lin1                : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
target_lin2                : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## ヒント

<details>
    <summary>clip to expand</summary>

- 動的なファイルのコピーには [templateモジュール](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html) を使用します
- 配布対象となるファイルは、 [Jinja2テンプレート](https://jinja.palletsprojects.com/en/2.11.x/) という形式で記述します  
     Ansible以外にも、例えば有名なWebアプリケーションフレームワークでにおいて動的にhtmlを生成する部分などで使用されている文法です  
     [このあたり](https://jinja.palletsprojects.com/en/2.11.x/templates/) が利用者向けのドキュメントです。  
     少々わかりにくい面もあるので、 [てくなべブログ](https://tekunabe.hatenablog.jp/entry/2019/03/03/ansible_template_intro) も参考にすると良いでしょう
- hostsファイルの作成時は、「ターゲットノード以外のIPアドレス」を参照する必要がでてくると思いますが、  
  このような場合、 [マジック変数](https://docs.ansible.com/ansible/2.9_ja/user_guide/playbooks_variables.html#magic-variables-and-hostvars) が利用できます
    - 今回の場合は、 `hostvars` や `groups` を使えばやりたいことが実現できそうです
        - 「`hostvars` にどんな値が入っているのか分からない！」という場合は、テンプレートに変数をそのまま出してみたり、debugモジュールを用いることで中身を確認してみましょう

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
├── hosts.j2
├── inventory
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
        src: ./hosts.j2
        dest: /etc/hosts
```

#### hosts.j2

```
127.0.0.1       localhost

{% for host in groups["all"] %}
  {{- host }} {{ hostvars[host]["ansible_host"] }}
{% endfor %}
```

[raw file](./answer/)  

### 解説

- hosts.j2は以下でも正解と言いたいところですが、「サーバ台数の変化」に対応することができない点が気がかりです。

    ```jinja2
    127.0.0.1       localhost

    target_lin1 {{ hostvars["target_lin1"]["ansible_host"] }}
    target_lin2 {{ hostvars["target_lin2"]["ansible_host"] }}
    ```

- 解答に示したj2テンプレートであれば、サーバ台数が変化した場合でもインベントリファイルにホストの情報を追加するだけで対応できます
    - 実PJで利用するにあたっては、汎用性を確保できる一方でテンプレートの内容が複雑になる点も考慮し、メンバーのスキルに応じてどこまで自動化するか検討しましょう

</details>

## Navigation

前：[1-6. ファイルコピー(copyモジュール)](1-6_copy-module/README.md)  
次：[1-8. ロールを使ってみよう](1-8_role/README.md)

[Top](../README.md)  
