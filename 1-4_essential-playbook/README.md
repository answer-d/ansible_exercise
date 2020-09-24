# 1-4. 具体的な処理を実行してみよう

## 問題

Playbookの雛形ができたので、構築や運用のための本格的な実装を始めようと思います。  
OSの基本設定としてやるべきことはたくさんありますが、手始めにPJで使うworkディレクトリを作成してみます。  
Playbookにタスクを追加し、以下の仕様のディレクトリを作成する処理を実装して下さい。  

|item|parameter|
|:--|:--|
|パス|`/nec_work/`|
|パーミッション|`777`|

作成したPlaybookの実行前後でad-hocコマンドを使って "ls -l /"を実行し、ディレクトリ作成がされたかどうかを確認してみると動作のイメージが付きやすいと思います。  
このようなad-hocコマンドはでディレクトリを参照できます。  

```console
ansible -i inventory -m command -a "ls -l /" linux
```

## 実行結果例

### 初回実行

```console
$ ansible-playbook -i inventory setup.yml

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
changed: [target_lin1]
changed: [target_lin2]

PLAY RECAP ***************************************************************************************
target_lin1                : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
target_lin2                : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### 2回目実行( `-v` 付き)

- `-v` をPlaybook実行時の引数として渡すと、実行結果を細かく表示してくれます(loglevelを上げているようなイメージです)

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
ok: [target_lin2] => {"changed": false, "gid": 0, "group": "root", "mode": "0777", "owner": "root", "path": "/nec_work", "secontext": "unconfined_u:object_r:default_t:s0", "size": 6, "state": "directory", "uid": 0}

PLAY RECAP ***************************************************************************************
target_lin1                : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
target_lin2                : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## ヒント

<details>
    <summary>clip to expand</summary>

- ディレクトリの作成にはfileモジュールが使用できます  
  <https://docs.ansible.com/ansible/latest/modules/file_module.html>
- 実行対象ノードへの接続に使用している認証情報は一般ユーザ(ansibleユーザ)です。そのため、このままではルートパーティション直下にディレクトリを作成する権限が無く、処理が失敗してしまいます
    - becomeという仕組みで権限昇格しながらモジュールを実行可能です、以下を参考に実装してみてください  
      <https://docs.ansible.com/ansible/2.9_ja//user_guide/become.html>
- fileモジュールのパラメータであるmodeに設定する値は文字列型( `"` でくくる)とスカラ型(くくらない)で設定内容が変化するため、注意が必要です。モジュールのドキュメントを良く読み、適切なパラメータを設定して下さい
    - くくる/くくらないのどちらかが正解というわけではなく、どちらのパターンでも適切な記述をすれば望みの状態にすることができます

</details>

## 解答

<details>
    <summary>clip to expand</summary>

### コード

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
```

[raw file](./answer/)  

### 解説

- `mode: 777` だと、パーミッションが意図したとおりに設定されません
    - `mode: 0777` とすることで、スカラ型でも望みの状態に持っていけます
- fileモジュールを使用せず、commandモジュールやshellモジュールでmkdirコマンドを発行することでも同じ処理は実現できます
    - が、これらのモジュールは単体では冪等性(何度実行しても同じ結果になる)を保証していないため、fileモジュールの使用を推奨します
    - command系のモジュールを使用して冪等性を保ちたい場合、Playbook内で開発者が頑張って冪等な処理を実装する必要があります
        - `mkdirコマンドは冪等ですか？` `どのようなコマンドラインなら冪等になりますか？` 等の問いに答えを出す必要があり、労力が必要です
    - command系モジュールは「なんでもできるけど調整が大変」ということで、最後の手段的な雰囲気で捉えておくと良いです
        - 行いたい処理を実装してくれているモジュールが存在する場合は、冪等性の確保をモジュール内で行ってくれている可能性が高いです。なるべく目的に合ったモジュールを使うようにしましょう

</details>

## Navigation

前：[1-3. 変数を使ってみよう](../1-3_variable/README.md)  
次：[1-5. 変数を使ったサーバごとの処理](../1-5_host-vars/README.md)  

[Top](../README.md)  
