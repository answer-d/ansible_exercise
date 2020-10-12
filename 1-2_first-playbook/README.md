# 1-2. はじめてのPlaybook

## 問題

なにはともあれ、まずはPlaybookを作成することにしました。  
以下の実行結果例のように、実行開始を画面に出力するPlaybookを作成して下さい。  

## 実行結果例

```console
$ ansible-playbook -i inventory setup.yml

PLAY [linuxグループセットアップ] ***************************************************************************

TASK [Gathering Facts] ***************************************************************************
ok: [target_lin1]
ok: [target_lin2]

TASK [実行開始通知] ************************************************************************************
ok: [target_lin1] => {
    "msg": "Playbookの実行を開始します"
}
ok: [target_lin2] => {
    "msg": "Playbookの実行を開始します"
}

PLAY RECAP ***************************************************************************************
target_lin1                : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
target_lin2                : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## ヒント

<details>
    <summary>clip to expand</summary>

- Playbookの全体像は以下のような形になります。★部分を埋めてみましょう  

    ```yaml
    ---
    - name: linuxグループセットアップ
      hosts: ★
      tasks:
        - ★
    ```

- Playbook実行結果に文字列を出力する場合は、debugモジュールを使用します  
  <https://docs.ansible.com/ansible/latest/modules/debug_module.html>
- `ansible debug` 」などで検索することで、日本語で書かれた情報も見つけることができます  
  OSSを扱う場合、ググり能力は非常に大切です、積極的にググっていきましょう
- Playbook実行時にエラーが発生した場合は、エラーとなった処理に出ているログの `msg` を確認すると原因が書いてあることがあります
- なお環境は異なりますが、以下のハンズオンドキュメントがPlaybookの基礎を学ぶにあたっておおいに参考になります  
  [Ansible Workshop - Ansible for Red Hat Enterprise Linux](https://github.com/ansible/workshops/blob/master/exercises/ansible_rhel/README.ja.md)

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
- name: linuxグループセットアップ
  hosts: linux
  tasks:
    - name: 実行開始通知
      debug:
        msg: Playbookの実行を開始します
```

[回答例の実ファイルはこちら](./answer/)  

### 解説

- Playbookはダッシュ記号 `-` の3連打からはじまります
    - Ansibleというよりはyamlの決まりです、なくても動きますが決まりなのでなるべく書きましょう
    - 余談ですが、yamlでは `---` がデータ構造の区切りを意味するため、以下のように「 `---` で区切って1ファイル内に複数のyamlデータを定義する」みたいな書き方が可能です  

        ```yaml
        ---
        hoge: hogehoge
        ---
        fuga: fugafuga
        ```

- yamlのインデントは半角スペースで行います、tabではエラーになります
- 今回実装したPlaybookに含まれる構成要素はざっと以下のような感じです

    |キー|意味|
    |:--|:--|
    |name|PlayやTaskの名前の定義、実行時に出力される|
    |hosts|タスクを実行する対象のホストグループやホスト。今回はlinuxグループを実行対象に指定している|
    |tasks|Ansibleモジュールを呼び出し、必要なオプションを渡すことで実行される操作のリスト。<br>モジュールに与えるオプションは、モジュール名の下の行に **段を下げて** 記述する|

- nameタグは無くても動きます。が、実行結果の可視性を高く保つためになるべく付けることをオススメします
- debugモジュールが受け付けるオプションの一覧はこちらを参照して下さい  
    <https://docs.ansible.com/ansible/latest/modules/debug_module.html>
    - 今回はこの中の `msg` というパラメータを使用しています

</details>

## Navigation

前：[1-1. 環境の確認](../1-1_setup/README.md)  
次：[1-3. 変数を使ってみよう](../1-3_variable/README.md)  

[Top](../README.md)  
