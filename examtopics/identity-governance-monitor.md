# Entra
## Entra ID
### ユーザー管理に必要なロール
- アカウント作成: **User Administrator**
- パスワードリセット: **Password Administrator** または **Helpdesk Administrator**
  - Helpdesk Administrator は以下の権限を有する
    - パスワードをリセットする
    - ユーザーを強制的にサインアウトさせる
    - サービスリクエストを管理する
    - サービス正常性を監視する

## Microsoft ID プラットフォーム
- 認証(Authentication: *AuthN*)は**身元を証明すること**, 認可(Authorization: *AuthZ*)は**権限を付与すること**
- Microsoft ID プラットフォームでは認可に **OAuth 2.0**, 認証に **OpenID Connect(OIDC)** が使用される
- 以下の認可/認証の組み合わせもサポートされる
  - OAuth & SAML
  - OpenID Connect & SAML

### B2B コラボレーション
- 自社のアプリケーションを外部ユーザーと共有できる
- **Entra ID のゲストアカウントが必要**
- RBAC で権限の付与が可能
- 外部ユーザーが既存の資格情報を使用できる


### アプリケーションの委任アクセス
- アプリケーションがメール, 予定表などの(サインインが必要な)データソースにアクセスするためには**委任**が必要

### アプリケーションに対するロールの追加とロールの付与
- アプリケーションのロールを宣言し, そのロールを他のアプリに割り当てることができる
  - App1 に対する Writer ロールを作成し, App2 に対してそのロールを付与するイメージ
  - ロールの宣言は Entra 管理センター > アプリの登録 > **アプリロール** から追加可能
  - アプリケーションへのロールの割り当ては, **API のアクセス許可**から設定
- [アプリケーションにアプリ ロールを追加してトークンで受け取る](https://learn.microsoft.com/ja-jp/entra/identity-platform/howto-add-app-roles-in-apps)

### シングルテナントアプリにサインインできるユーザーの変更
- 異なるアカウントをサポートするようにアプリケーションの登録を変更する
- シングルテナントアプリをマルチテナントに変換する
  - 登録をマルチテナントに更新
  - リクエストを `/common` に送信するようにコードを変更する
    - ユーザーのテナントを特定する

## Microsoft Entra ID Governance
- ID ライフサイクルの管理, アクセスのライフサイクルの管理, 管理に使用される特権アクセスのセキュリティ保護 などをサポート
- 具体的なサービス例:
  - **エンタイトルメント管理**
    - 内部および**外部**のユーザーのアクセスを効率的に管理
    - B2B で外部ユーザーを招待するにはメールアドレスを知っている必要があるが, エンタイトルメント管理で自動化できる
  - **プロビジョニング**
    - 複数のシステム間でデジタル ID の一貫性を確保するプロセス
    - 以下の3つに大別される:
      - 人事主導のプロビジョニング
        - 人事システム内の情報に基づいてオブジェクトを作成する(Cloud HR から Entra ID)
      - **アプリのプロビジョニング**
        - アプリケーションのユーザーの ID とロールを自動作成すること
        - Entra から SaaS アプリケーション (Dropbox, Salesforce, ServiceNow, ...) へのプロビジョニング
        - System for Cross-Domain Identity Management(SCIM): 異なるドメイン間でアイデンティティ情報を管理するためのプロトコル
      - ディレクトリ間のプロビジョニング
        - Active Directory -> Entra ID へのプロビジョニング
  - **PIM(Privileged Identity Management)**
    - 時間ベースおよび承認ベースのロールのアクティブ化を提供. 具体的な機能は以下:
      - 時間ベース
        - Just-In-Time 特権アクセスの提供
        - 期限付きアクセス権をリソースに割り当てる
      - 承認ベース
        - 特権ロールのアクティブ化に承認を要求する
        - 特権ロールのアクティブ化時に通知を受ける
        - ロールのアクティブ化に多要素認証を強制する
        - アクティブ化に理由を要求する
        - アクセスレビューの実施
  - **アクセスレビュー**
  - ID ライフサイクル管理


# ガバナンス
## Policy
### Azure Policy の割り当て範囲
- 管理グループ, サブスクリプション, リソースグループ単位で割り当て可能
- **リソースごとに割り当てることはできない**

### コンプライアンスデータの取得
- アクティビティログ(Log Analytics の `AzureActivity`)からアラートの作成などが可能
  - 最大90日間保存される

#### オンデマンド
- Azure CLI, Azure PowerShell, REST API または GitHub Action for Azure Policy Compliance Scan で開始できる
  - CLI: `az policy state trigger-scan`
  - PowerShell: `Start-AzPolicyComplianceScan`


# Monitor
## 仮想マシンからイベントとパフォーマンスカウンターを取得する
- データの収集には収集ルールの作成が必要
  - リソースの追加
    - Azure Monitor エージェントがインストールされていないリソースにはインストールする
    - プライベートリンクも使用できる
  - **データ収集エンドポイント**を有効にする

## Log Analytics のコミットメントレベル
- あらかじめ使用するデータ量を購入しておくと, 超過した分もその料金で課金される
- ワークスペースが 200 GB/日のコミットメントレベルにあり, 1日に 300 GB を取り込む場合, 200 GB/日コミットメント レベルの 1.5 ユニットとして課金される
- [コミットメントレベル](https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/cost-logs#commitment-tiers)


## Synthetic Transaction
- 複数のコンテナでホストされているアプリに対して, コンポーネント間のトラフィックを監視するには "synthetic transaction monitoring" を使用する
  - Application Insight でサポートされている


# セキュリティ
## Key Vault
### キーとシークレットの違い
- キー: セキュリティに使用する情報を格納する(暗号化キー)
- シークレット: アプリケーションに必要な機密データ(API キーなど)

### App Service と Key Vault の連携
- アプリ設定や接続文字列の値として Key Vault のシークレットを使用できる