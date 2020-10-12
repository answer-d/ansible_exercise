# 1-3. 変数を使ってみよう

## 問題

前問のPlaybookは実行対象ホスト分、全く同じメッセージが表示されていました。  
これでは少々冗長なので、メッセージの内容を少し意味のあるものにしようと思います。  
Playbookを修正し、メッセージ内に対象サーバのIPアドレスを出力するようにして下さい。  

## 実行結果例

```console
$ ansible-playbook -i inventory setup.yml

PLAY [linuxグループセットアップ] ***************************************************************************

TASK [Gathering Facts] ***************************************************************************
ok: [target_lin2]
ok: [target_lin1]

TASK [実行開始通知] ************************************************************************************
ok: [target_lin1] => {
    "msg": "192.168.250.237に対してPlaybookの実行を開始します"
}
ok: [target_lin2] => {
    "msg": "192.168.159.104に対してPlaybookの実行を開始します"
}

PLAY RECAP ***************************************************************************************
target_lin1                : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
target_lin2                : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## ヒント

<details>
    <summary>clip to expand</summary>

- 変数の参照方法を調べて、debugモジュールの出力内容に上記の変数を入れ込んでみましょう
    - 以下が公式情報です  
      <https://docs.ansible.com/ansible/2.9_ja/user_guide/playbooks_variables.html#jinja2>
- IPアドレスは対象サーバでコマンドを実行するなどして取得せずとも、既にインベントリファイルに書いてある変数を利用できます
    - 具体的には、ansible_hostという変数に対象サーバのIPアドレスが入っています。インベントリファイルを確認してみましょう

</details>

## 解答

<details>
    <summary>clip to expand</summary>

### コード

#### ファイル構成

```plain
    setup.yml
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
```

[回答例の実ファイルはこちら](./answer/)  

### 解説

- Playbookでは、変数名を二重中括弧で囲むことで変数を参照します

    ```yaml
    {{ variable1 }}
    ```

- 変数やその値は、インベントリ、追加ファイル、コマンドラインなどのさまざまな場所で定義できます
    - 変数の定義可能な場所と書き方のイメージ、優先順位については以下の記事を参考にして下さい  
      <https://qiita.com/answer_d/items/b8a87aff8762527fb319>

</details>

## Navigation

前：[1-2. はじめてのPlaybook](../1-2_first-playbook/README.md)  
次：[1-4. 具体的な処理を実行してみよう](../1-4_essential-playbook/README.md)  

[Top](../README.md)  
