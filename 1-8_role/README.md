# 1-8. ロールを使ってみよう

## 問題

ここまでで作成してきたように、Playbookを単一1つのファイルで管理することは可能です。  
しかしそのうち、作成したPlaybookを分割して再利用可能にしたり、汎用化したりしたいと思うようになります。  
Ansibleではこの実現のため、ロールというものを用意しています。  

ロールは基本的にはタスクのリストを分割する機能です。  
加えて以下のような便利な機能がいくつか付与されています。  

- そのロールで使いたい変数を定義する機能
- メタ情報の付与(ロール間の依存関係など)
- ハンドラ(特定のタスクを実行したときにのみ実行されるタスクの定義方法)の自動読み込み
- `copy` モジュールや `template` モジュールでコピーする元ファイルの格納場所を提供

※ 詳細 → <https://docs.ansible.com/ansible/2.9_ja/user_guide/playbooks_reuse_roles.html>

## 実行結果例

```console
```

## ヒント

<details>
    <summary>clip to expand</summary>

- ロールを使うにはにはAnsibleにより定められたディレクトリ構造に従ってファイルを作成する必要があります、まずはそれを調べましょう
- 今回の場合、ロール内に作成すべきフォルダは `tasks` および `templates` あたりになるでしょう

</details>

## 解答

<details>
    <summary>clip to expand</summary>

### コード

#### ファイル構成

```plain
```

#### setup.yml

```yaml
```

#### hosts.j2

```
```

[raw file](./answer/)  

### 解説

- ほげほげら

</details>

## Navigation

前：[1-7. ファイルコピー(templateモジュール)](1-7_template-module/README.md)  
次：なし (基礎編はこれで終わりです)

[Top](../README.md)  
