# security-ci

複数のGitHubリポジトリから呼び出せる、共通セキュリティチェック用のGitHub Actionsです。Android / Flutter / Web / Backend / IaCプロジェクトを想定しています。

## チェック内容

| Job | ツール | 対象 |
| --- | --- | --- |
| secrets | Gitleaks | コミット履歴中のシークレット |
| sast | Semgrep | ソースコードの静的解析 |
| sca | Trivy | 依存関係の既知脆弱性（HIGH / CRITICAL、修正可能なもの） |
| iac | Checkov | Terraform / Kubernetes / CloudFormation / Helm / Serverless |
| actions-security | zizmor | GitHub Actionsの設定上の問題 |

各Jobは独立して実行され、入力値で個別に無効化できます。IaCとGitHub Actionsのチェックは対象ファイルがなければスキップします。

## 導入

利用側リポジトリに `.github/workflows/security.yml` を作成します。

```yaml
name: Security

on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  security:
    uses: tiny0Xy9/security-ci/.github/workflows/security.yml@main
    with:
      run_secrets: true
      run_sast: true
      run_sca: true
      run_iac: true
      run_actions: true
      fail_on_findings: true
```

コピー可能な例: [examples/consumer.yml](examples/consumer.yml)

**本番運用では `@main` の代わりにレビュー済みのコミットSHAに固定してください。** タグやブランチは後から更新されるため、サプライチェーン攻撃への耐性が弱くなります。ワークフロー内で利用する外部Actionも本番導入前にコミットSHAへ固定することを推奨します。

## 入力値

| Input | Default | 説明 |
| --- | --- | --- |
| `run_secrets` | `true` | Gitleaks |
| `run_sast` | `true` | Semgrep |
| `run_sca` | `true` | Trivy |
| `run_iac` | `true` | Checkov |
| `run_actions` | `true` | zizmor |
| `fail_on_findings` | `true` | 検出時にJobを失敗させる。falseは試験導入向け |

## 注意事項

- 呼び出し元のGitHub Actions設定で、外部の再利用可能ワークフローの実行が許可されている必要があります。
- 公開リポジトリのワークフローを非公開リポジトリから呼び出せます。ソースコードがこのリポジトリにコピーされるわけではありませんが、GitHub Actionsランナー上でスキャンされます。
- Semgrepは自動設定のCommunityルールを使用します。全ての言語・脆弱性を網羅するものではありません。
- Trivyはファイルシステムの依存関係スキャンです。コンテナイメージのビルド後スキャンは別途追加が必要です。
- Checkovの対象検出は簡易判定です。Kubernetes YAMLのみのリポジトリなどでは手動でJobを調整してください。
- Gitleaks Actionの利用条件・ライセンス要件は利用環境に応じて確認してください。
- 追加ツールのバージョンは現状一部固定されていません。厳密な再現性が必要な場合はバージョン固定してください。
- MobSFとOWASP ZAPは、対象アプリのビルド成果物や検証用URLが必要なため、この初期版には含めていません。
- このCIはセキュリティレビューや動的診断の代替にはなりません。

## 開発

`.github/workflows/self-test.yml` はこのリポジトリ自体に共通ワークフローを適用します。初期導入では `fail_on_findings: false` とし、結果を確認してから厳格化してください。
