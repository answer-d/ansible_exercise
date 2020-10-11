# 1-6. ファイルコピー(copyモジュール)

## 問題

名前解決はhostsによって行うようです。  
以下のホスト名とIPアドレスを解決可能なとなるようhostsファイルを **静的に** (=ファイルの内容が固定的に) 作成し、  
`target_lin1` と `target_lin2` の `/etc/hosts` として配布して下さい。  

## 実行結果例

```console
PLAY [linuxグループセットアップ] ****************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************************************************************************************************************************
ok: [target_lin2]
ok: [target_lin1]

TASK [実行開始通知] *************************************************************************************************************************************************************************************************************************************************************
ok: [target_lin1] => {
    "msg": "target_lin1に対してPlaybookの実行を開始します"
}
ok: [target_lin2] => {
    "msg": "target_lin2に対してPlaybookの実行を開始します"
}

TASK [workディレクトリ作成] *******************************************************************************************************************************************************************************************************************************************************
ok: [target_lin1] => {"changed": false, "gid": 0, "group": "root", "mode": "0777", "owner": "root", "path": "/nec_work", "size": 4096, "state": "directory", "uid": 0}
ok: [target_lin2] => {"changed": false, "gid": 0, "group": "root", "mode": "0777", "owner": "root", "path": "/nec_work", "size": 4096, "state": "directory", "uid": 0}

TASK [AP用ディレクトリ作成] ********************************************************************************************************************************************************************************************************************************************************
ok: [target_lin1] => {"changed": false, "gid": 1000, "group": "ansible", "mode": "0770", "owner": "ansible", "path": "/nec_work/hoge", "size": 4096, "state": "directory", "uid": 1000}
ok: [target_lin2] => {"changed": false, "gid": 1000, "group": "ansible", "mode": "0770", "owner": "ansible", "path": "/nec_work/poyo", "size": 4096, "state": "directory", "uid": 1000}

TASK [hosts配布] ************************************************************************************************************************************************************************************************************************************************************
changed: [target_lin2] => {"changed": true, "checksum": "1384b95a0ff0555dbcc12fbef4913e7893c7c746", "dest": "/etc/hosts", "gid": 0, "group": "root", "md5sum": "8d1b6c0211c11608bc317f85ff3ca5a3", "mode": "0644", "owner": "root", "size": 83, "src": "/home/ansible/.ansible/tmp/ansible-tmp-1601806296.78-611-677951760005/source", "state": "file", "uid": 0}
changed: [target_lin1] => {"changed": true, "checksum": "1384b95a0ff0555dbcc12fbef4913e7893c7c746", "dest": "/etc/hosts", "gid": 0, "group": "root", "md5sum": "8d1b6c0211c11608bc317f85ff3ca5a3", "mode": "0644", "owner": "root", "size": 83, "src": "/home/ansible/.ansible/tmp/ansible-tmp-1601806296.71-607-245377195888988/source", "state": "file", "uid": 0}

PLAY RECAP ****************************************************************************************************************************************************************************************************************************************************************
target_lin1                : ok=5    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
target_lin2                : ok=5    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## ヒント

<details>
    <summary>clip to expand</summary>

- 静的なファイルのコピーには [copyモジュール](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html) を使用します
- hostsファイルの元ネタをターゲットサーバからsshやscpなどで取得してファイルを作成しましょう

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
├── hosts
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
      copy:
        src: ./hosts
        dest: /etc/hosts
```

#### hosts

```
127.0.0.1       localhost

192.168.250.237 target_lin1
192.168.159.104 target_lin2
```

[raw file](./answer/)  

### 解説

- hostsファイルの所有者はrootですので、 `become` で特権昇格が必要です

</details>

## Navigation

前：[1-5. 変数を使ったサーバごとの処理](../1-5_host-vars/README.md)  
次：[1-7. ファイルコピー(templateモジュール)](../1-7_template-module/README.md)  

[Top](../README.md)  
