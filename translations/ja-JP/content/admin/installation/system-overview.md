---
title: システムの概要
intro: '{{ site.data.variables.product.prodname_ghe_server }} は仮想化アプライアンス内に含まれる {{ site.data.variables.product.prodname_dotcom }} のお客様の Organization のプライベートなコピーで、オンプレミスあるいはクラウド上でホストされ、お客様が設定およびコントロールできます。'
redirect_from:
  - /enterprise/admin/installation/system-overview
versions:
  enterprise-server: '*'
---

### ストレージアーキテクチャ

{{ site.data.variables.product.prodname_ghe_server }} は、2 つのストレージボリュームを必要とします。1 つは*ルートファイルシステム*パス (`/`) にマウントされるもので、もう 1 つは*ユーザファイルシステム*パス (`/data/user`) にマウントされるものです。 このアーキテクチャは、動作するソフトウェアの環境を永続的なアプリケーションデータから分離することによって、アップグレード、ロールバック、リカバリの手続きをシンプルにします。

ルートファイルシステムは、配布されているマシンイメージに含まれています。 ルートファイルシステムにはベースのオペレーティングシステムと {{ site.data.variables.product.prodname_ghe_server }} アプリケーション環境が含まれています。 ルートファイルシステムは、一過性のものとして扱われなければなりません。 ルートファイルシステム上にあるデータは、すべて将来の {{ site.data.variables.product.prodname_ghe_server }} リリースへのアップグレード時に置き換えられます。

ルートファイルシステムには以下が含まれます:
  - カスタムの認証局 (CA) 証明書 (*/usr/local/share/ca-certificates*)
  - カスタムのネットワーク設定
  - カスタムのファイアウォール設定
  - レプリケーションの状態

ユーザファイルシステムには、以下のようなユーザ設定とデータが含まれます:
  - Git リポジトリ
  - データベース
  - 検索インデックス
  - {{ site.data.variables.product.prodname_pages }} サイトで公開されたコンテンツ
  - {{ site.data.variables.large_files.product_name_long }} からの大きなファイル
  - pre-receive フック環境

### デプロイメントの選択肢

{{ site.data.variables.product.prodname_ghe_server }} は単一の仮想アプライアンスとしても、High Availability 構成としてもデプロイできます。 詳細は「[High Availability 用に {{ site.data.variables.product.prodname_ghe_server }} を設定する](/enterprise/{{ currentVersion }}/admin/guides/installation/configuring-github-enterprise-server-for-high-availability/)」を参照してください。

数万人の開発者がいる組織の場合、{{ site.data.variables.product.prodname_ghe_server }} クラスタリングも有益かもしれません。 詳しい情報については「[クラスタリングについて](/enterprise/{{ currentVersion }}/admin/guides/clustering/about-clustering)」を参照してください。

### データのリテンションとデータセンターの冗長性

{% danger %}

{{ site.data.variables.product.prodname_ghe_server }} を本番環境で使う前に、バックアップとシステム災害復旧計画をセットアップしておくことを強くおすすめします。 詳しい情報については、「[アプライアンスでのバックアップの設定](/enterprise/{{ currentVersion }}/admin/guides/installation/configuring-backups-on-your-appliance)」を参照してください。

{% enddanger %}

{{ site.data.variables.product.prodname_ghe_server }} には、[{{ site.data.variables.product.prodname_enterprise_backup_utilities }}](https://github.com/github/backup-utils) でのオンラインおよびインクリメンタルバックアップのサポートが含まれています。 インクリメンタルスナップショットは、オフサイトや地理的に離れたストレージのために長距離を経てセキュアなネットワークリンク（SSH管理ポート）経由で取ることができます。 スナップショットは、プライマリデータセンターにおける災害時のリカバリにおいて、新たにプロビジョニングされたアプライアンスにネットワーク経由でリストアできます。

ネットワークバックアップに加えて、アプライアンスがオフラインになっているかメンテナンスモードになっている間に、ユーザストレージボリュームのAWS（EBS）やVMWareのディスクスナップショットがサポートされています。 サービスレベルの要求が定期的なオフラインメンテナンスを許せるものであれば、定期的なボリュームのスナップショットは、{{ site.data.variables.product.prodname_enterprise_backup_utilities }}のネットワークバックアップの低コストで複雑さの低い代替になります。

詳しい情報については、「[アプライアンスでのバックアップの設定](/enterprise/{{ currentVersion }}/admin/guides/installation/configuring-backups-on-your-appliance)」を参照してください。

### セキュリティ

{{ site.data.variables.product.prodname_ghe_server }} は、お客様のインフラストラクチャ上で動作する仮想アプライアンスで、ファイアウォール、IAM、監視、VPNなどの既存のセキュリティ情報管理により統御されます。 {{ site.data.variables.product.prodname_ghe_server }} を使うと、クラウドベースのソリューションから生じる、規制の遵守に関する問題の回避に役立ちます。

{{ site.data.variables.product.prodname_ghe_server }} には、追加のセキュリティ機能も含まれています。

- [オペレーティングシステム、ソフトウェア、パッチ](#operating-system-software-and-patches)
- [ネットワークのセキュリティ](#network-security)
- [アプリケーションのセキュリティ](#application-security)
- [外部サービスおよびサポートへのアクセス](#external-services-and-support-access)
- [暗号化通信](#encrypted-communication)
- [ユーザおよびアクセス権限](#users-and-access-permissions)
- [認証](#authentication)
- [監査およびアクセスのログ取得](#audit-and-access-logging)

#### オペレーティングシステム、ソフトウェア、パッチ

{{ site.data.variables.product.prodname_ghe_server }} は、必要なアプリケーションとサービスのみを備える、カスタマイズされた Linux を実行します。 {{ site.data.variables.product.prodname_dotcom }} は、標準的な製品リリースサイクルの一環として、アプライアンスのコアオペレーティングシステムに対するパッチを管理します。 パッチは、{{ site.data.variables.product.prodname_dotcom }} アプライアンスの機能、安定性、および重大でないセキュリティ問題に対処するものです。 また、{{ site.data.variables.product.prodname_dotcom }} は、必要に応じて標準的なリリースサイクル外でも重大なセキュリティパッチを提供します。

#### ネットワークのセキュリティ

{{ site.data.variables.product.prodname_ghe_server }} の内部ファイアウォールは、アプライアンスのサービスへのネットワークアクセスを制限します。 アプライアンスが機能するために必要なサービスだけが、ネットワークを通じて利用できます。 詳しい情報については、「[ネットワークポート](/enterprise/{{ currentVersion }}/admin/guides/installation/network-ports)」を参照してください。

#### アプリケーションのセキュリティ

{{ site.data.variables.product.prodname_dotcom }} の、アプリケーションのセキュリティチームは、{{ site.data.variables.product.prodname_ghe_server }} も含めた {{ site.data.variables.product.prodname_dotcom }} 製品に対し、脆弱性評価、ペネトレーションテスト、およびコードレビューをフルタイムで重点的に取り組んでいます。 また、{{ site.data.variables.product.prodname_dotcom }} は、{{ site.data.variables.product.prodname_dotcom }} 製品の特定時点におけるセキュリティ評価を行なうため、外部セキュリティ企業と契約しています。

#### 外部サービスおよびサポートへのアクセス

{{ site.data.variables.product.prodname_ghe_server }} は、お客様のネットワークから外部サービスにアクセスすることなしに運用できます。 また、メール配信、外部モニタリング、およびログ転送のため、外部サービスとのインテグレーションを有効にすることも可能です。 詳しい情報については、「[通知のためのメール設定](/enterprise/{{ currentVersion }}/admin/user-management/configuring-email-for-notifications)」、「[外部モニタリングのセットアップ](/enterprise/{{ currentVersion }}/admin/installation/setting-up-external-monitoring)」、および「[ログの転送](/enterprise/{{ currentVersion }}/admin/installation/log-forwarding)」を参照してください。

トラブルシューティングデータを手動で収集し、{{ site.data.variables.contact.github_support }} に送信できます。 詳しい情報については、「[{{ site.data.variables.contact.github_support }} へのデータ提供](/enterprise/{{ currentVersion }}/admin/enterprise-support/providing-data-to-github-support)」を参照してください。

#### 暗号化通信

{{ site.data.variables.product.prodname_dotcom }} は、{{ site.data.variables.product.prodname_ghe_server }} がお客様の社内ファイアウォールの背後で動作するよう設計しています。 回線を介した通信を保護するため、Transport Layer Security (TLS) を有効化するようお勧めします。 {{ site.data.variables.product.prodname_ghe_server }} は、HTTPS トラフィックに対して、2048 ビット以上の商用 TLS 証明書をサポートしています。 詳しい情報については、「[TLSの設定](/enterprise/{{ currentVersion }}/admin/installation/configuring-tls)」を参照してください。

デフォルトでは、Git によるリポジトリへのアクセスと管理目的との両方で、アプライアンスは Secure Shell (SSH) アクセスも提供します。 詳しい情報については、「[SSH について](/enterprise/user/articles/about-ssh)」および「[管理シェル (SSH) にアクセスする](/enterprise/{{ currentVersion }}/admin/installation/accessing-the-administrative-shell-ssh)」を参照してください。

#### ユーザおよびアクセス権限

{{ site.data.variables.product.prodname_ghe_server }} は、3 種類のアカウントを提供しています。

- `admin` Linux ユーザアカウントは、ファイルシステムやデータベースへの直接的なアクセスを含め、基底のオペレーティングシステムに対して限定的にアクセスできます。 このアカウントには、少数の信頼できる管理者がアクセスできるようにすべきで、SSH を介してアクセスできます。 詳しい情報については、「[管理シェル (SSH) にアクセスする](/enterprise/{{ currentVersion }}/admin/installation/accessing-the-administrative-shell-ssh)」を参照してください。
- アプライアンスのウェブアプリケーション内のユーザアカウントは、自らのデータ、および他のユーザや Organization が明示的に許可したあらゆるデータにフルアクセスできます。
- アプライアンスのウェブアプリケーション内サイト管理者は、高レベルのウェブアプリケーションおよびアプライアンスの設定、ユーザおよび Organization のアカウント設定、ならびにリポジトリデータを管理できるユーザアカウントです。

{{ site.data.variables.product.prodname_ghe_server }} のユーザ権限に関する詳しい情報については「[GitHub 上のアクセス権限](/enterprise/user/articles/access-permissions-on-github)」を参照してください。

#### 認証

{{ site.data.variables.product.prodname_ghe_server }} は、4 つの認証方式を提供しています。

- SSH 公開鍵認証は、Git によるリポジトリへのアクセスと、管理シェルアクセスの両方を提供します。 詳しい情報については、「[SSH について](/enterprise/user/articles/about-ssh)」および「[管理シェル (SSH) にアクセスする](/enterprise/{{ currentVersion }}/admin/installation/accessing-the-administrative-shell-ssh)」を参照してください。
- HTTP クッキーを用いたユーザ名とパスワードによる認証では、ウェブアプリケーションのアクセスおよびセッションの管理、そして任意で 2 要素認証 (2FA) を提供します。 詳しい情報については、「[ビルトイン認証の利用](/enterprise/{{ currentVersion }}/admin/user-management/using-built-in-authentication)」を参照してください。
- LDAP サービス、SAML アイデンティティプロバイダ (IdP)、またはその他互換性のあるサービスを用いた外部 LDAP、SAML、および CAS 認証は、ウェブアプリケーションへのアクセスを提供します。 詳しい情報については、「[GitHub Enterprise Server インスタンスでユーザを認証する](/enterprise/{{ currentVersion }}/admin/user-management/authenticating-users-for-your-github-enterprise-server-instance)」を参照してください。
- OAuth および個人アクセストークンは、外部クライアントとサービスの両方に対して、Git リポジトリデータおよび API へのアクセスを提供します。 For more information, see "[Creating a personal access token](/github/authenticating-to-github/creating-a-personal-access-token)."

#### 監査およびアクセスのログ取得

{{ site.data.variables.product.prodname_ghe_server }} は、従来型オペレーティングシステムおよびアプリケーションの両方のログを保存します。 また、アプリケーションは詳細な監査およびセキュリティログも記録し、{{ site.data.variables.product.prodname_ghe_server }} はこれを永続的に保存します。 `syslog-ng` プロトコルにより、両タイプのログをリアルタイムで複数の宛先に転送できます。 詳しい情報については、「[ログの転送](/enterprise/{{ currentVersion }}/admin/installation/log-forwarding)」を参照してください。

アクセスログと監査ログには、以下のような情報が含まれています。

##### アクセスログ

- ブラウザと API アクセスの両方の、ウェブサーバーの完全なログ
- Git、HTTPS、および SSH プロトコルを介した、リポジトリデータへのアクセスの完全なログ
- HTTPS および SSH を介した、管理アクセスのログ

##### 監査ログ

- ユーザのログイン、パスワードのリセット、2 要素認証のリクエスト、メール設定の変更、ならびに許可されたアプリケーションおよび API への変更
- ユーザアカウントやリポジトリのアンロックなどの、サイト管理者のアクション
- リポジトリのプッシュイベント、アクセス許可、移譲、および名前の変更
- チームの作成および破棄を含む、Organization のメンバーシップ変更

### {{ site.data.variables.product.prodname_ghe_server }} のオープンソース依存性

使用しているアプライアンスのバージョンの {{ site.data.variables.product.prodname_ghe_server }} における依存対象の完全なリストは、それぞれのプロジェクトのライセンスと合わせて `http(s)://HOSTNAME/site/credits` で見ることができます。

依存関係と関連するメタデータの完全なリストと合わせて、Tarball 群はアプライアンス上にあります。
- すべてのプラットフォームに共通の依存関係は `/usr/local/share/enterprise/dependencies-<GHE version>-base.tar.gz` にあります。
- プラットフォームに固有の依存関係は `/usr/local/share/enterprise/dependencies-<GHE version>-<platform>.tar.gz` にあります。

依存対象とメタデータの完全なリストとともにTarball群も`https://enterprise.github.com/releases/<version>/download.html`にあります。

### 参考リンク

- [{{ site.data.variables.product.prodname_ghe_server }} のトライアルをセットアップする](/articles/setting-up-a-trial-of-github-enterprise-server)
- [{{ site.data.variables.product.prodname_ghe_server }} インスタンスをセットアップする](/enterprise/{{ currentVersion }}/admin/guides/installation/setting-up-a-github-enterprise-server-instance)