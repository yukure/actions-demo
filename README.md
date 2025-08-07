# actions-demo

このリポジトリは、GitHub ActionsとAWS CloudFormationを組み合わせたCI/CDパイプラインのdemo用リポジトリです。

## 概要

AWS S3バケットの管理を通じて、ステージング環境と本番環境への自動デプロイメントワークフローを実装しています。

## ワークフロー構成

### 1. Release Workflow (`.github/workflows/release.yml`)
**トリガー**: `main`ブランチへのpushまたはタグ(`v*`)のpush

このワークフローは本格的なデプロイメントプロセスを実行します：

- **ステージング環境へのデプロイ**
  - AWS CloudFormationを使用して`StagingStack`をデプロイ
  - `staging.yml`テンプレートを使用してS3バケット(`yukure-staging-bucket`)を作成・更新

- **本番環境へのデプロイ**
  - ステージング環境のデプロイ成功後に実行
  - `ProductionStack`をデプロイ
  - `production.yml`テンプレートを使用してS3バケット(`yukure-production-bucket`)を作成・更新

### 2. Pull Request Workflow (`.github/workflows/pull-request.yml`)
**トリガー**: Pull Requestの作成・更新

このワークフローはプルリクエスト時の変更内容をプレビューします：

- **Change Set作成・プレビュー機能**
  - ステージング環境とプロダクション環境それぞれに対してChange Setを作成
  - CloudFormationの変更内容をプルリクエストにコメントとして自動投稿
  - Change Setは確認後に自動削除される

- **通知機能**
  - Slackへの通知機能を内蔵

### 3. Create Release Note Workflow (`.github/workflows/create-release-note.yml`)
**トリガー**: mainブランチへのプルリクエストのマージ、または手動実行

自動リリースノート生成機能：

- **自動バージョニング**
  - 日時ベースのバージョン番号生成（YYYYMMDDHHMM形式）
  - 人が読みやすい形式のタイトル生成（YYYY-MM-DD_HH:MM形式）

- **自動リリース作成**
  - GitHubの自動生成機能を使用してリリースノートを作成
  - タグの自動作成とプッシュ

## インフラストラクチャ

### CloudFormationテンプレート

- **`staging.yml`**: ステージング環境用S3バケット設定
- **`production.yml`**: 本番環境用S3バケット設定

両環境とも以下の特徴を持ちます：
- AES256による暗号化
- 適切なパブリックアクセス制御
- バケットの保持ポリシー（削除・更新時の保護）

## セキュリティ

- **AWS IAMロール**: GitHub ActionsからのOIDC認証を使用
- **最小権限の原則**: 必要最小限の権限のみを付与
- **リージョン**: ap-northeast-1（東京）を使用

## 使用方法

1. **開発フロー**: プルリクエストを作成して変更内容をプレビュー
2. **リリースフロー**: mainブランチにマージして自動デプロイ実行
3. **リリースノート**: マージ時に自動でリリースノートを生成

このデモンストレーションを通じて、GitHub ActionsとAWS CloudFormationを組み合わせた実践的なCI/CDパイプラインの構築方法を学ぶことができます。