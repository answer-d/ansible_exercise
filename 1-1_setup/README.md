# 1-1. 環境の確認

1. [環境について](#環境について)
2. [準備](#準備)

## 環境について

本ハンズオンは `セルフラーニング環境` に存在する以下のサーバで行います。  
IPアドレスは自環境のものを使用して下さい。  

- Ansible実行サーバ (RHEL7.6)
    - Exastro ITA
- 構築対象サーバ (RHEL7.6)
    - Target Lin#1
    - Target Lin#2

## 準備

**※ OneNote参照**  

- ログイン確認
- code-serverセットアップ
- インベントリファイル作成  
  以降、以下のインベントリファイルが作成されている想定となります(回答例にも含まれないので注意)  

    ```ini
    [linux]
    target_lin1 ansible_host=<Target Lin#1のIPアドレス>
    target_lin2 ansible_host=<Target Lin#2のIPアドレス>

    [linux:vars]
    ansible_user=ansible
    ansible_password=ansible
    ```

- ad-hocコマンドの実行

## Navigation

[次：1-2. はじめてのPlaybook](../1-2_first-playbook/README.md)  

[Top](../README.md)  
