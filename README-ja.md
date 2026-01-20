# Red Hat Quay PoC Ansible Playbook

このAnsible Playbookは、[Proof of Concept - Deploying Red Hat Quay](https://docs.redhat.com/ja/documentation/red_hat_quay/3.16/html-single/proof_of_concept_-_deploying_red_hat_quay/index) に基づき、Red Hat Quay 3.16のProof of Concept (PoC) 環境を構築します。

**注意**: 本Playbookは、Red Hat QuayのProof of Concept環境を構築するためのものです。本Playbookを用いて本番環境に構築することは推奨しません。

## 概要

以下のコンポーネントをデプロイします：
- **Podman**: コンテナランタイム
- **PostgreSQL 15**: Quay用データベース (pg_trgm拡張を含む)
- **Redis 6**: ビルドログおよびユーザーイベント用
- **Red Hat Quay 3.16**: コンテナレジストリ本体

## 前提条件

- **Ansible**: コントロールノードにAnsibleがインストールされていること
- **RHEL 9 or RHEL10**: ターゲットノード（またはローカルホスト）がRHEL 9またはRHEL 10であること
- **Red Hat Subscription**: ターゲットノードがRed Hatに登録されていること
- **Registry Credentials**: `registry.redhat.io` にアクセス可能なRed Hatアカウント情報

## セットアップ

### 1. インベントリの設定

`inventory` ファイルを編集し、デプロイ先のホストを指定してください。デフォルトでは `localhost` が設定されています。

```ini
[quay_servers]
localhost ansible_connection=local
```

### 2. 変数の設定

`group_vars/all.yml` を編集し、必要な各変数を設定します。特に以下のレジストリ認証情報は必須です。

**推奨**: 環境変数として設定する

```bash
export REDHAT_REGISTRY_USERNAME="your-rh-username"
export REDHAT_REGISTRY_PASSWORD="your-rh-password"
```

または、`group_vars/all.yml` を直接編集して設定することも可能です（非推奨）。

```yaml
registry_username: "your_username"
registry_password: "your_password"
```

その他、`group_vars/all.yml` 内の以下の変数も環境に合わせて確認・変更してください。

- `quay_base_dir`
- `server_hostname`
- `server_ip`
- `quay_admin_password`
- `quay_admin_email`
- `secret_key` / `database_secret_key`

## 実行方法

必要となるAnsibleコレクションをインストールします：

```bash
ansible-galaxy collection install containers.podman
```

Playbookを実行します：

```bash
ansible-playbook -i inventory playbook.yml
```

*注意: Playbook内で `become: true` を使用しているため、通常 `ansible-playbook` コマンド自体に `sudo` を付与する必要はありません。ただし、実行ユーザーが sudo 昇格にパスワードを必要とする場合は、`-K` (または `--ask-become-pass`) オプションを追加してください。*

## 確認方法

デプロイ完了後、以下の手順で動作を確認できます。

1. **コンテナの確認**:
   ```bash
   sudo podman ps
   ```
   `quay`, `redis`, `postgresql-quay` コンテナが稼働していることを確認します。

2. **ブラウザでのアクセス**:
   `https://<server_hostname>` にアクセスし、Quayのログイン画面が表示されることを確認します。
   ※自己署名証明書（または無効な証明書）の警告が出る場合がありますが、PoCのため無視して進めてください。

3. **ログイン**:
   `group_vars/all.yml` で設定した管理者ユーザー（デフォルト: `quayadmin`）は `SUPER_USERS` 配列に追加されていますが、初回起動時にデータベースが空のため、まずは「Create Account」から同名のアカウントを作成するか、構成によっては初期化ウィザードが表示される場合があります。

## ディレクトリ構成

- `inventory`: インベントリファイル
- `group_vars/`: 変数定義
- `roles/`:
  - `prerequisites`: Podmanインストール、Firewall設定など
  - `database`: PostgreSQLセットアップ
  - `redis`: Redisセットアップ
  - `quay`: Quayの設定生成とデプロイ
