# (社内向け) Ansibleハンズオンネタ

## 前提

あなたは現在担当中のPJで、IaC(Ansible)によるLinuxサーバ群の自動構築を行うことになりました。  
IaCには以下の要件が求められています。  

- 対象サーバはリソース需要により増加/減少する可能性があり、サーバ台数が増加した場合でも楽に対応可能な、拡張性の高い自動化を行いたい
- 実装したIaCは構築フェーズだけではなく、運用フェーズでも利用が想定されているため、繰り返し何度も実行可能な(=冪等な)実装が求められている
    - つまり、「特定の初期状態からしか実行できない」や「1回しか実行できない」といった成果物では不十分となる
- 運用フェーズではオペミス減少を目的としてドライラン(チェックモード)での実行も想定しており、なるべくドライランが動作する状態を保つ必要がある
    - ただしこれはストレッチ目標であり、現状必ずしも達成する必要はない

## Section 1 - 基礎編

- [1-1. 環境の確認](1-1_setup/README.md)
- [1-2. はじめてのPlaybook](1-2_first-playbook/README.md)
- [1-3. 変数を使ってみよう](1-3_variable/README.md)
- [1-4. 具体的な処理を実行してみよう](1-4_essential-playbook/README.md)
- [1-5. 変数を使ったサーバごとの処理](1-5_host-vars/README.md)
- [1-6. ファイルコピー(copyモジュール)](1-6_copy-module/README.md)
- [1-7. ファイルコピー(templateモジュール)](1-7_template-module/README.md)
- [1-8. ロールを使ってみよう](1-8_role/README.md)

## Section 2 - 中級編

- [2-1. 環境の確認](2-1_setup/README.md)
- [2-2. パッケージの管理](2-2_package-management/README.md)
- [2-3. 条件分岐](2-3_when/README.md)
- [2-4. ハンドラ](2-4_handler/README.md)
- [2-5. フィルタ](2-5_filter/README.md)
- [2-6. 複雑なループ](2-6_complicated-loop/README.md)
- [2-7. ブロック](2-7_block/README.md)
- [2-8. プラグイン](2-8_plugin/README.md)
- [2-9. ダイナミックインベントリ](2-9_dynamic-inventory/README.md)
- [2-10. デバッガ](2-10_debugger/README.md)
- [2-11. ドライラン](2-11_dryrun/README.md)
- [2-12. Ansibleでテストを自動化する](2-12_test-automation/README.md)
- [2-13. 機密情報の暗号化](2-13_vault/README.md)
- [2-14. 実行戦略](2-14_strategy/README.md)
- [2-15. 委譲](2-15_delegation/README.md)
- [2-X. あとがき](2-X_postscript/README.md)

## Section 3 - 発展編

- [3-1. 環境の確認](3-1_setup/README.md)
- [3-2. プレイブックにおける冪等性の担保](3-2_keep-idempotence/README.md)
- [3-3. ansible-lintによるコードの静的解析](3-3_ansible-lint/README.md)
- [3-4. moleculeによるロールのテスト](3-4_molecule/README.md)
