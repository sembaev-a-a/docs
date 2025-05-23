# クラスター モード

<PluginInfo licenseBundled="enterprise" plugins="pubsub-adapter-redis,lock-adapter-redis"></PluginInfo>

NocoBase は v1.6.0 バージョンからアプリケーションのクラスター モードでの実行をサポートしています。アプリケーションがクラスター モードで実行されると、複数のインスタンスとマルチコア モードを使用して、アプリケーションの同時アクセス処理性能を向上させることができます。

## システムアーキテクチャ

![20241231010814](https://static-docs.nocobase.com/20241231010814.png)

### アーキテクチャ コンポーネント

現在のクラスター モードはアプリケーション インスタンスのみに対応しており、分散アーキテクチャ内の他のシステム コンポーネントは、異なるチームの運用ニーズに応じて、現在の制約に従ってチームの運用担当者が自由に選択します。

#### アプリケーションクラスター

アプリケーションクラスターは NocoBase アプリのインスタンスの集合であり、各インスタンスは独立したノードであるか、単一のマシン上でマルチコア モードで実行される複数のアプリケーションプロセスであるか、またはその両方を混合して使用できます。

#### データベース

現在のクラスター モードはアプリケーション インスタンスのみに対応しているため、データベースは一時的に単一ノードのみをサポートしており、主従関係などのデータベース アーキテクチャが必要な場合は、自身でミドルウェアを通じて実現し、NocoBase アプリに対して透過的であることを保証する必要があります。

#### キャッシュ、同期メッセージ、分散ロック

NocoBase クラスター モードは、クラスター間の通信と調整を実現するためにキャッシュ、同期メッセージ、分散ロックなどのミドルウェアに依存する必要があります。現在、初歩的に Redis を使用して対応する機能のミドルウェアのサポートが行われています。

#### 負荷分散

NocoBase クラスター モードは、リクエストの分配、およびアプリケーション インスタンスのヘルスチェックとフェイルオーバーを実現するためにロードバランサを介する必要があります。この部分は、チームの運用ニーズに応じて自由に選択および設定します。

## デプロイ手順

### インフラストラクチャの準備

#### 負荷分散

自建の Nginx を例として、設定ファイルに以下の内容を追加します：

```
upstream myapp {
    # ip_hash; # セッション保持に使用でき、開けると同一クライアントからのリクエストは常に同じバックエンドサーバに送信されます。
    server 172.31.0.1:13000; # 内部ノード1
    server 172.31.0.2:13000; # 内部ノード2
    server 172.31.0.3:13000; # 内部ノード3
}

server {
    listen 80;

    location / {
        # 定義された upstream を使用して負荷分散
        proxy_pass http://myapp;
        # ... 他の設定
    }
}
```

これはリクエストを異なるサーバーノードにリバースプロキシで分配して処理します。
他のクラウドサービスプロバイダーが提供する負荷分散ミドルウェアについては、具体的なサービスプロバイダーが提供する設定ドキュメントを参照してください。

#### Redis サービス

クラスター内部ネットワーク（または k8s）内で、Redis サービスを起動します。または、異なる機能（キャッシュ、同期メッセージ、分散ロック）ごとにそれぞれ Redis サービスを有効にします。

#### ローカルストレージ（必要に応じて）

ローカルストレージを使用している場合は、マルチノード モードでは、クラウドディスクをマウントしてローカルストレージディレクトリとして使用し、マルチノードによる共有アクセスをサポートします。さもなければ、ローカルストレージは自動的に同期されず、正常には使用できません。

ローカルストレージを使用しない場合、アプリケーション起動後に、クラウドサービスに基づくファイルストレージスペースをデフォルトのファイルストレージスペースとして設定し、アプリケーションのロゴ（または他のファイル）をクラウドストレージスペースに移動する必要があります。

### 関連プラグイン準備

| 機能 | プラグイン |
| --- | --- |
| キャッシュ | 内蔵 |
| 同期メッセージ | @nocobase/plugin-pubsub-adapter-redis |
| 分散ロック | @nocobase/plugin-lock-adapter-redis |

:::info{title=ヒント}
単一ノード モードのアプリケーションと同様に、商用プラグインサービスプラットフォームに関連する環境変数を設定すれば、アプリケーション起動後に自動的に関連するプラグインがダウンロードされます。
:::

### 環境変数の設定

基本的な環境変数のほか、以下の環境変数はすべてのノードで設定する必要があり、かつ一致させておく必要があります。

#### アプリケーションキー

ユーザーがログインする際に JWT Token を作成するためのキーで、ランダムな文字列を使用することを推奨します。

```ini
APP_KEY=
```

#### マルチコア モード

```ini
# PM2 マルチコア モードを有効にする
# CLUSTER_MODE=max # デフォルトでは無効、手動で設定する必要があります
```

#### キャッシュ

```ini
# キャッシュアダプタ、クラスター モードでは redis を指定する必要があります（デフォルトはメモリ）
CACHE_DEFAULT_STORE=redis

# Redis キャッシュアダプタの接続アドレス、明示的に入力する必要があります
CACHE_REDIS_URL=
```

#### 同期メッセージ

```ini
# Redis 同期アダプタの接続アドレス、デフォルトは redis://localhost:6379/0
PUBSUB_ADAPTER_REDIS_URL=
```

#### 分散ロック

```ini
# ロックアダプタ、クラスター モードでは redis を指定する必要があります（デフォルトはメモリローカルロック）
LOCK_ADAPTER_DEFAULT=redis

# Redis ロックアダプタの接続アドレス、デフォルトは redis://localhost:6379/0
LOCK_ADAPTER_REDIS_URL=
```

:::info{title=ヒント}
通常、関連するアダプタは同じ Redis インスタンスを使用できますが、潜在的なキーの衝突問題を回避するために異なるデータベースを使用することをお勧めします。

```ini
CACHE_REDIS_URL=redis://localhost:6379/0
PUBSUB_ADAPTER_REDIS_URL=redis://localhost:6379/1
LOCK_ADAPTER_REDIS_URL=redis://localhost:6379/2
```
:::

#### 内蔵プラグインの有効化

```ini
# 有効にする内蔵プラグイン
APPEND_PRESET_BUILT_IN_PLUGINS=lock-adapter-redis,pubsub-adapter-redis
```

### アプリケーションの起動

アプリケーションを初めて起動する際は、最初にノードの一つを起動し、プラグインのインストールと有効化を待ってから、他のノードを起動する必要があります。

## アップグレードまたはメンテナンス

NocoBaseのバージョンをアップグレードしたり、プラグインを有効/無効にする必要がある場合は、このプロセスを参考にしてください。

:::warning{title=注意}
クラスターの生産環境では、プラグイン管理やバージョンアップグレードなどの機能を慎重に使用するか禁止する必要があります。

NocoBaseは現在、クラスター版のオンラインアップグレードを実装していないため、データの整合性を確保するために、アップグレード中はサービスを一時停止する必要があります。
:::

### 現在のサービスを停止する

すべてのNocoBaseアプリケーションインスタンス、Redisなどのミドルウェアを停止し、負荷分散のトラフィックを503ステータスページに転送します。

### データのバックアップ

アップグレード前に、異常が発生するのを防ぐためにデータベースのデータをバックアップすることを強くお勧めします。

### バージョンを更新する

[NocoBaseアプリケーションイメージのバージョンを更新するためのDockerアップグレード](../upgrading/docker-compose.md)を参照してください。

### サービスを起動する

1. 依存しているミドルウェア（Redis）を再起動します。
2. クラスター内のノードの一つを起動し、更新が完了して正常に起動するのを待ちます。
3. 機能が正しいか確認します。異常があった場合、トラブルシューティングで解決できないなら、前のバージョンにロールバックします。
4. 他のノードを起動します。
5. 負荷分散のトラフィックをアプリケーションクラスターに転送します。
