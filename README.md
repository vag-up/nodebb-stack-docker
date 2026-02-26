# Docker環境構築（NodeBB + MongoDB + Nginx + Redis）

Vagrantを使用してUbuntu 24.04上にDocker環境を自動構築するプロジェクトです。
Ansibleを使用して必要なソフトウェアのインストールと設定を行い、BBSシステム [NodeBB](https://nodebb.org/) を実行するための環境を構築します。

## 環境構成

- **OS**: Ubuntu 24.04 (bento/ubuntu-24.04)
- **Docker**: geerlingguy.docker ロールによるインストール
- **タイムゾーン**: Asia/Tokyo
- **SSL/TLS**: 自己署名証明書によるHTTPS接続

### Dockerコンテナ構成

| コンテナ | 説明 |
|---------|------|
| NodeBB | BBSシステム（公式Dockerイメージ、ポート4567） |
| Nginx | リバースプロキシ/Webサーバー（HTTPS対応） |
| MongoDB | データベースサーバー |
| Redis | キャッシュ/セッションサーバー |

## 前提条件

- [VirtualBox](https://www.virtualbox.org/) がインストールされていること
- [Vagrant](https://www.vagrantup.com/) がインストールされていること
- 十分なメモリ（最低2GB）と空きディスク容量があること

## ディレクトリ構成

```
.
├── README.md             # このファイル
├── Vagrantfile           # Vagrant設定ファイル
└── playbooks/            # Ansibleプレイブック
    ├── main.yml              # メインプレイブック
    ├── requirements.yml      # 必要なロールの定義
    ├── vars/                 # 変数定義
    │   └── main.yml              # メイン変数ファイル
    ├── tasks/                # タスク定義
    │   ├── japanese.yml          # 日本語環境設定
    │   ├── redis.yml             # Redisコンテナ構築
    │   ├── mongodb.yml           # MongoDBコンテナ構築
    │   ├── nginx.yml             # Nginxコンテナ構築
    │   └── nodebb.yml            # NodeBBコンテナ構築
    └── containers/           # コンテナ設定
        ├── mongodb/              # MongoDB用Dockerfile等
        ├── nginx/                # Nginx用設定ファイル等
        ├── nodebb/               # NodeBB用setup.json、install-data（日本語初期データ）、language（日本語リソース）
        └── redis/                # Redis用Dockerfile等
```

## IPアドレス設定

仮想マシンには固定IPアドレス `192.168.33.10` が設定されます。
必要に応じて `Vagrantfile` の `config.vm.network` の設定を変更してください。

## 起動手順

1. リポジトリをクローンする
2. 以下のコマンドを実行してVirtualBox上に環境を構築

```bash
vagrant up
```

## 接続方法

環境構築後、以下のいずれかの方法で仮想マシンに接続できます。

1. Vagrantから接続:
```bash
vagrant ssh
```

2. SSHで直接接続:
```bash
ssh vagrant@192.168.33.10
```
※デフォルトパスワード: vagrant

### rootユーザーへの切り替え
接続後、以下のコマンドでrootユーザーに切り替えることができます。
```bash
sudo su -
```

## Dockerコンテナの確認

環境構築後、仮想マシン内で以下のコマンドでコンテナの状態を確認できます。

```bash
docker ps                    # 実行中のコンテナ一覧
docker logs nodebb           # NodeBBコンテナのログ
docker logs nginx            # Nginxコンテナのログ
docker logs mongodb          # MongoDBコンテナのログ
docker logs redis            # Redisコンテナのログ
```

## データベース設定

MongoDBの初期設定は以下の通りです（`playbooks/vars/main.yml` で変更可能）:

| 項目 | 値 |
|------|-----|
| rootユーザー | root |
| rootパスワード | root_password |
| データベース名 | nodebb |
| NodeBBユーザー名 | nodebb |
| NodeBBパスワード | nodebb_password |

接続URI: `mongodb://nodebb:nodebb_password@mongodb:27017/nodebb`

## カスタマイズ

- `playbooks/vars/main.yml` を編集することで、コンテナ名やDB設定を変更できます
- `playbooks/main.yml` を編集することで、追加のパッケージやタスクを追加できます
- `Vagrantfile` の `vb.memory` を編集して、仮想マシンのメモリ割り当てを変更できます

## SSL/HTTPS接続

この環境では、NginxがHTTPS（ポート443）でリッスンし、自己署名SSL証明書を使用してセキュアな接続を提供します。

### HTTPからHTTPSへの自動リダイレクト

HTTPポート（80）へのアクセスは自動的にHTTPS（443）にリダイレクトされます。

### ブラウザからのアクセス時の注意

自己署名証明書を使用しているため、ブラウザで初回アクセス時に「安全ではありません」という警告が表示されます。
これは開発環境では正常な動作です。以下の手順で続行できます：

1. ブラウザで `https://192.168.33.10` にアクセス
2. 「詳細設定」または「Advanced」をクリック
3. 「安全ではないページに移動」または「Proceed to 192.168.33.10 (unsafe)」をクリック

## NodeBB 初回セットアップ

初回起動時、NodeBBは `config.json` が存在しないため **Webインストーラー** を起動します。

### 手順

1. ブラウザで `https://192.168.33.10` にアクセス
2. Webインストーラーが表示されたら、以下の項目を入力：
   - **サイトURL**: `https://192.168.33.10` を正確に入力（アクセスするURLと一致させること）
   - **データベース**: MongoDB を選択
   - **MongoDBホスト**: `mongodb`（コンテナ名）
   - **MongoDBポート**: `27017`
   - **MongoDBユーザー名**: `nodebb`
   - **MongoDBパスワード**: `nodebb_password`（`vars/main.yml` の `main_db_password` の値）
   - **MongoDBデータベース名**: `nodebb`
   - **Redisホスト**: `redis`（セッション/キャッシュ用）
3. セットアップ完了後、管理者アカウントを作成
4. フォーラムの利用を開始

### 注意事項

- サイトURLは、実際にアクセスするURLと一致してください。誤ったURLを入力すると、リダイレクトやアセット読み込みに問題が発生します。
- セットアップ完了後、`config.json` が `/docker/nodebb/config/` に自動生成され、以降は自動起動します。

### 日本語初期データ

初回セットアップ時に、カテゴリ（お知らせ、フリートーク、ブログ、コメント・フィードバック）やウェルカム投稿を日本語で作成するように設定されています。設定ファイルは `playbooks/containers/nodebb/install-data/` にあり、ボリュームマウントでコンテナ内の `install/data/` を上書きしています。日本語UIリソースは `playbooks/containers/nodebb/language/ja/` にあり、プロビジョニング時に `/docker/nodebb/language` へコピーしてからボリュームマウントされます。

**注意**: この設定は**初回セットアップ時のみ**有効です。既にNodeBBをセットアップ済みの場合は反映されません。日本語の初期データでやり直すには、`/docker/nodebb/config/config.json` を削除し、MongoDBの `nodebb` データベースを削除してから再セットアップしてください。

## トラブルシューティング

### ネットワーク接続の問題
ネットワーク設定に問題が発生した場合は、`Vagrantfile` の IPアドレスを
使用環境に合わせて変更してください。

### 仮想マシンの起動に失敗する場合
VirtualBoxの設定や競合を確認し、必要に応じてVirtualBoxを再起動してください。

### プロビジョニングを再実行する場合
```bash
vagrant provision
```

### NodeBBの再セットアップが必要な場合
`/docker/nodebb/config/config.json` を削除してから再起動すると、Webインストーラーが再度表示されます。

### 「No write permission for directory /opt/config」エラーが出る場合
NodeBBコンテナは UID 1001 で動作します。既存環境でこのエラーが出る場合は、以下で権限を修正してから `vagrant provision` を再実行してください。

```bash
vagrant ssh
sudo chown -R 1001:1001 /docker/nodebb/build /docker/nodebb/uploads /docker/nodebb/config
exit
vagrant provision
```
