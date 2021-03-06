---
title: 手順1基本的な DirectAccess インフラストラクチャを計画する
description: このトピックは、「Windows Server 2016 用はじめにウィザードを使用して単一の DirectAccess サーバーを展開する」の一部です。
manager: brianlic
ms.custom: na
ms.prod: windows-server
ms.reviewer: na
ms.suite: na
ms.technology: networking-da
ms.tgt_pltfrm: na
ms.topic: article
ms.assetid: ''
ms.author: pashort
author: shortpatti
ms.openlocfilehash: 6f4c727dc8f7905502d47119bd0e911537e827aa
ms.sourcegitcommit: 6aff3d88ff22ea141a6ea6572a5ad8dd6321f199
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/27/2019
ms.locfileid: "71404877"
---
# <a name="step-1-plan-the-basic-directaccess-infrastructure"></a>手順1基本的な DirectAccess インフラストラクチャを計画する
1台のサーバーでの基本的な DirectAccess 展開の最初の手順は、展開に必要なインフラストラクチャを計画することです。 このトピックでは、インフラストラクチャの計画手順を説明します。  
  
|タスク|説明|  
|----|--------|  
|ネットワークのトポロジと設定を計画する|エッジまたはネットワークアドレス変換 \(NAT\) デバイスまたはファイアウォール\)の内側に DirectAccess サーバー \(を配置する場所を決定し、IP アドレス指定とルーティングを計画します。|  
|ファイアウォールの要件を計画する|DirectAccess がエッジ ファイアウォールを通過できるようにすることを計画します。|  
|証明書の要件を計画する|DirectAccess は、クライアント認証に Kerberos または証明書を使用できます。 この基本的な DirectAccess 展開では、Kerberos プロキシが自動的に構成され、認証は Active Directory の資格情報を使って行われます。|  
|DNS の要件を計画する|DirectAccess サーバー、インフラストラクチャ サーバー、およびクライアント接続の DNS 設定を計画します。|  
|Active Directory を計画する|ドメイン コントローラーと Active Directory の要件を計画します。|  
|グループ ポリシー オブジェクトを計画する|組織で必要な GPO とその GPO の作成または編集方法を決定します。|  
  
計画タスクは特定の順序で実行する必要はありません。  
  
## <a name="bkmk_1_1_Network_svr_top_settings"></a>ネットワークトポロジと設定を計画する  
  
### <a name="plan-network-adapters-and-ip-addressing"></a>ネットワーク アダプターと IP アドレス指定を計画する  
  
1.  使用するネットワーク アダプターのトポロジを決定します。 DirectAccess は次のいずれかを使ってセットアップできます。  
  
    -   2つのネットワークアダプターがあり、一方のネットワークアダプターがインターネットに接続されていて、もう一方が境界ネットワークに接続されていて、もう一方のネットワークアダプターが内部ネットワークに接続されている。ネットワーク.  
  
    -   NAT デバイスの背後に1つのネットワークアダプターがある場合-DirectAccess サーバーは NAT デバイスの内側にインストールされ、1つのネットワークアダプターが内部ネットワークに接続されます。  
  
2.  IP アドレス指定の要件を特定します。  
  
    DirectAccess は、IPv6 と IPsec を組み合わせて、DirectAccess クライアント コンピューターと企業内部ネットワークとの間にセキュリティで保護された接続を確立します。 ただし、DirectAccess は、IPv6 インターネットへの接続または内部ネットワーク上でのネイティブ IPv6 サポートを必ずしも必要とはしません。 代わりに、IPv6 移行テクノロジを自動的に構成して使用し、IPv4 インターネット \(6to4、Teredo、IP\-HTTPS\) 経由で IPv6 トラフィックをトンネリングするようにします。また、IPv4\-だけで、NAT64 または ISATAP \(を使用します。\) このような移行テクノロジの概要については、次のリソースを参照してください。  
  
    -   [IPv6 移行テクノロジ](https://technet.microsoft.com/library/bb726951.aspx)  
  
    -   [Ip-https トンネリングプロトコル仕様](https://msdn.microsoft.com/library/dd358571(PROT.10).aspx)  
  
3.  次の表に従って、必要なアダプターとアドレス指定を構成します。 1つのネットワークアダプターを使用して NAT デバイスの背後に配置する場合は、 **[内部ネットワークアダプター]** 列のみを使用して IP アドレスを構成します。  
  
    ||外部ネットワーク アダプター|内部ネットワーク アダプター<sup>1</sup>|ルーティングの要件|  
    |-|--------------|--------------------|------------|  
    |IPv4 イントラネットおよび IPv4 インターネット|次を構成します。<br /><br />-適切なサブネットマスクを持つ1つの静的パブリック IPv4 アドレス。<br />-インターネットファイアウォールまたはローカルインターネットサービスプロバイダーの既定のゲートウェイ IPv4 アドレス \(ISP\) ルーター。|次を構成します。<br /><br />適切なサブネット マスクを持つ IPv4 イントラネット アドレスが。<br />-イントラネット名前空間の特定の DNS サフィックスを\-接続。 DNS サーバーは内部インターフェイスにも構成する必要があります。<br />-イントラネットインターフェイスに既定のゲートウェイを構成しないでください。|内部 IPv4 ネットワーク上のすべてのサブネットに到達できるように DirectAccess サーバーを構成するには、次のようにします。<br /><br />1. イントラネット上のすべての場所の IPv4 アドレス空間を一覧表示します。<br />2. **route add \-p**または**netsh interface ipv4 add route**コマンドを使用して、ipv4 アドレス空間を静的ルートとして DirectAccess サーバーの ipv4 ルーティングテーブルに追加します。|  
    |IPv6 インターネットおよび IPv6 イントラネット|次を構成します。<br /><br />-ISP によって提供される自動構成されたアドレス構成を使用します。<br />- **Route print**コマンドを使用して、ISP ルーターを指す既定の ipv6 ルートが ipv6 ルーティングテーブルに存在することを確認します。<br />-ISP ルーターとイントラネットルーターが RFC 4191 で説明されている既定のルーター基本設定を使用し、ローカルイントラネットルーターよりも高い既定の基本設定を使用しているかどうかを確認します。 どちらについても使用している場合は、既定のルートに他の構成は必要ありません。 ISP ルーターの高度な基本設定によって、DirectAccess サーバーの既定の IPv6 アクティブ ルートが IPv6 インターネットを示すことが保証されます。<br /><br />DirectAccess サーバーは IPv6 ルーターであるため、ネイティブ IPv6 インフラストラクチャがある場合は、インターネット インターフェイスからイントラネット上のドメイン コントローラーに到達することもできます。 この場合は、境界ネットワーク内のドメインコントローラーにパケットフィルターを追加して、DirectAccess サーバーのインターネット\-接続インターフェイスの IPv6 アドレスに接続できないようにします。|次を構成します。<br /><br />-既定の基本設定レベルを使用していない場合は、 **netsh interface ipv6 Set InterfaceIndex ignoredefaultroutes\=enabled**コマンドを使用して、イントラネットインターフェイスを構成します。 このコマンドを実行すると、イントラネット ルーターを示す既定のルートがそれ以上 IPv6 ルーティング テーブルに追加されないことが保証されます。 イントラネット インターフェイスの InterfaceIndex は、netsh interface show interface コマンドの表示で確認できます。|IPv6 イントラネットがある場合、IPv6 のすべての場所に到達できるように DirectAccess サーバーを構成するには、次のようにします。<br /><br />1. イントラネット上のすべての場所の IPv6 アドレス空間を一覧表示します。<br />2. **netsh interface ipv6 add route**コマンドを使用して、ipv6 アドレス空間を静的ルートとして DirectAccess サーバーの ipv6 ルーティングテーブルに追加します。|  
    |IPv4 インターネットおよび IPv6 イントラネット|DirectAccess サーバーは、IPv4 インターネット上の 6to4 リレーへの Microsoft 6to4 Adapter インターフェイスを使って、既定の IPv6 ルート トラフィックを転送します。 次のコマンドを使用して、ネイティブ IPv6 が企業\) ネットワークに展開されていない場合に使用される IPv4 インターネット \(で、Microsoft 6to4 リレーの IPv4 アドレス用に DirectAccess サーバーを構成できます。 netsh interface IPv6 6to4 set relay name\=192.88.99.1 state\=enabled コマンド。|||  
  
    > [!NOTE]  
    > 次の点に注意してください。  
    >   
    > 1.  パブリック IPv4 アドレスが割り当てられている DirectAccess クライアントは、6to4 移行テクノロジを使ってイントラネットに接続します。 DirectAccess クライアントが6to4 を使用して DirectAccess サーバーに接続できない場合は、IP\-HTTPS が使用されます。  
    > 2.  ネイティブ IPv6 クライアント コンピューターは、ネイティブ IPv6 経由で DirectAccess サーバーに接続できます。移行テクノロジを必要としません。  
  
### <a name="ConfigFirewalls"></a>ファイアウォールの要件を計画する  
エッジ ファイアウォールの背後に設置された DirectAccess サーバーが IPv4 インターネット上にあるときは、DirectAccess トラフィックについて次の例外が必要になります。  
  
-   6to4 トラフィック-IP プロトコル41の受信と送信。  
  
-   IP\-HTTPS-伝送制御プロトコル \(TCP\) 宛先ポート443、TCP 発信元ポート443送信。  
  
-   1 つのネットワーク アダプターを使って DirectAccess を展開し、ネットワーク ロケーション サーバーを DirectAccess サーバーにインストールする場合は、TCP ポート 62000 も除外する必要があります。  
  
    > [!NOTE]  
    > この除外は DirectAccess サーバー上です。 その他すべての例外はエッジ ファイアウォール上です。  
  
DirectAccess サーバーが IPv6 インターネット上にあるときは、DirectAccess トラフィックについて次の例外が必要になります。  
  
-   IP プロトコル 50  
  
-   UDP 宛先ポート 500 の受信と、UDP 発信元ポート 500 の送信。  
  
追加のファイアウォールを使用する場合は、次の内部ネットワーク ファイアウォール例外を DirectAccess トラフィックに適用します。  
  
-   ISATAP-プロトコル41の受信と送信  
  
-   すべての IPv4\/IPv6 トラフィックに対する TCP\/UDP  
  
### <a name="bkmk_1_2_CAs_and_certs"></a>証明書の要件を計画する  
IPsec の証明書の要件には、DirectAccess クライアント コンピューターと DirectAccess サーバーとの間で IPsec 接続を確立するときにクライアントによって使用されるコンピューター証明書と、DirectAccess クライアントとの IPsec 接続を確立するために DirectAccess サーバーによって使用されるコンピューター証明書が含まれます。 Windows Server 2012 R2 および Windows Server 2012 の DirectAccess の場合、これらの IPsec 証明書の使用は必須ではありません。 作業の開始ウィザードでは、証明書を要求せずに IPsec 認証を実行する Kerberos プロキシとして機能するように DirectAccess サーバーを構成します。
  
1.  **IP\-HTTPS サーバー**。 DirectAccess を構成すると、DirectAccess サーバーは、IP\-HTTPS web リスナーとして機能するように自動的に構成されます。 IP\-HTTPS サイトには web サイト証明書が必要です。クライアントコンピューターは、証明書の CRL\) サイト \(証明書失効リストに接続できる必要があります。 DirectAccess の有効化ウィザードでは、SSTP VPN 証明書の使用を試みます。 SSTP が構成されていない場合は、IP\-HTTPS の証明書がコンピューターの個人用ストアに存在するかどうかを確認します。 使用できるものがない場合は、自己\-署名入り証明書が自動的に作成されます。
  
2.  **ネットワーク ロケーション サーバー**: ネットワークロケーションサーバーは、クライアントコンピュータが企業ネットワークに配置されているかどうかを検出するために使用される web サイトです。 ネットワークロケーションサーバーには、web サイトの証明書が必要です。 DirectAccess クライアントは、その証明書の CRL サイトに接続できる必要があります。 リモートアクセスの有効化ウィザードでは、ネットワークロケーションサーバーの証明書がコンピューターの個人用ストアに存在するかどうかを確認します。 存在しない場合は、自己\-署名入り証明書が自動的に作成されます。
  
これらの各要素の認定要件を次の表にまとめます。  
  
|IPsec 認証|IP\-HTTPS サーバー|ネットワーク ロケーション サーバー|  
|------------|----------|--------------|  
|認証に Kerberos プロキシを使用しない場合に、内部 CA が DirectAccess サーバーとの IPsec 認証にクライアントをコンピューター証明書を発行するために必要|パブリック CA-パブリック CA を使用して IP\-HTTPS 証明書を発行することをお勧めします。これにより、CRL 配布ポイントを外部で使用できるようになります。|内部 CA-内部 CA を使用して、ネットワークロケーションサーバーの web サイト証明書を発行できます。 CRL 配布ポイントが内部ネットワークからいつでも利用できることを確認してください。|  
||内部 CA-内部 CA を使用して、IP\-HTTPS 証明書を発行できます。ただし、CRL 配布ポイントが外部で使用可能であることを確認する必要があります。|自己\-署名入り証明書-ネットワークロケーションサーバーの web サイトの自己\-署名入り証明書を使用できます。ただし、マルチサイト展開では、自己\-署名入り証明書を使用することはできません。|  
||自己\-署名入り証明書-IP\-HTTPS サーバーに自己\-署名された証明書を使用できます。ただし、CRL 配布ポイントが外部で使用可能であることを確認する必要があります。 自己\-署名入り証明書をマルチサイト展開で使用することはできません。||  
  
#### <a name="bkmk_website_cert_IPHTTPS"></a>IP\-HTTPS およびネットワークロケーションサーバーの証明書を計画する  
この目的で証明書をプロビジョニングする場合は、「[詳細設定を使用して単一の DirectAccess サーバーを展開する](../single-server-advanced/Deploy-a-Single-DirectAccess-Server-with-Advanced-Settings.md)」を参照してください。 使用できる証明書がない場合は、はじめにウィザードによって、自己\-署名入り証明書が自動的に作成されます。
  
> [!NOTE]
> IP\-HTTPS およびネットワークロケーションサーバー用の証明書を手動でプロビジョニングする場合は、証明書にサブジェクト名があることを確認します。 サブジェクト名がなくて代替名がある証明書は、DirectAccess ウィザードで受け付けられません。  
  
#### <a name="plan-dns-requirements"></a>DNS の要件を計画する  
DirectAccess 展開では、次のことについて DNS が必要です。  
  
-   **DirectAccess クライアント要求**。 内部ネットワークに配置されていない DirectAccess クライアント コンピューターからの要求を解決するためには、DNS が使用されます。 DirectAccess クライアントは、DirectAccess ネットワークロケーションサーバーへの接続を試行して、インターネットまたは企業ネットワーク上に配置されているかどうかを判断します。接続が成功した場合、クライアントはイントラネット上にあると判断され、DirectAccess は使用されず、クライアントの要求は、クライアントコンピューターのネットワークアダプターに構成されている DNS サーバーを使用して解決されます。 接続に失敗した場合、クライアントはインターネット上にあると見なされます。 DirectAccess クライアントは、名前解決ポリシーテーブル \(NRPT\) を使用して、名前要求を解決するときに使用する DNS サーバーを決定します。 クライアントが名前解決に DirectAccess DNS64 または代替の内部 DNS サーバーを使用するように指定することができます。 名前解決の実行時、NRPT が DirectAccess クライアントによって使用されて、要求の処理方法が特定されます。 クライアントは、内部 \/http:\/のように、FQDN または単一の\-ラベル名を要求します。 1つの\-ラベル名が要求の場合は、FQDN を作成するために DNS サフィックスが追加されます。 DNS クエリが NRPT 内のエントリに一致し、そのエントリに対して DNS4 またはイントラネット DNS サーバーが指定されている場合、そのクエリは指定されたサーバーを使って名前解決のために送信されます。 一致するエントリはあっても DNS サーバーが指定されていない場合は、除外規則を意味しているので、通常の名前解決が適用されます。  
  
    DirectAccess 管理コンソール内の NRPT に新しいサフィックスを追加するとき、そのサフィックスの既定の DNS サーバーを自動的に検出するには、 **[検出]** をクリックします。 自動検出のしくみは次のとおりです。  
  
    1.  企業ネットワークが IPv4\-ベース、または IPv4 と IPv6 の場合、既定のアドレスは、DirectAccess サーバーの内部アダプターの DNS64 アドレスです。  
  
    2.  企業ネットワークが IPv6\-基づいている場合、既定のアドレスは、企業ネットワーク内の DNS サーバーの IPv6 アドレスです。  
  
-   **インフラストラクチャサーバー**
  
    1.  **ネットワーク ロケーション サーバー**: DirectAccess クライアントは、内部ネットワークに配置されているかを判断するために、ネットワーク ロケーション サーバーに接続しようとします。 内部ネットワーク上のクライアントは、ネットワーク ロケーション サーバーの名前を解決できる必要がありますが、インターネットに配置されているときは名前を解決できないようにする必要があります。 このような処理が行われるように、既定では、ネットワーク ロケーション サーバーの FQDN が除外規則として NRPT に追加されます。 また、DirectAccess を構成するときは、次の規則が自動的に作成されます。  
  
        1.  ルート ドメインまたは DirectAccess サーバーのドメイン名の DNS サフィックス規則、および DirectAccess サーバー上に構成されたイントラネット DNS サーバーに対応する IPv6 アドレス。 たとえば、DirectAccess サーバーが corp.contoso.com ドメインのメンバーである場合、corp.contoso.com DNS サフィックスについて規則が作成されます。  
  
        2.  ネットワーク ロケーション サーバーの FQDN の除外規則。 たとえば、ネットワークロケーションサーバーの URL が https:\/\/nls.corp.contoso.com の場合、FQDN nls.corp.contoso.com の除外ルールが作成されます。  
  
        **IP\-HTTPS サーバー**。 DirectAccess サーバーは、IP\-HTTPS リスナーとして機能し、そのサーバー証明書を使用して IP\-HTTPS クライアントを認証します。 IP\-の HTTPS 名は、パブリック DNS サーバーを使用する DirectAccess クライアントによって解決できる必要があります。  
  
        **接続検証方法**。 DirectAccess は、DirectAccess クライアント コンピューターが内部ネットワークへの接続を確認するのに使用する既定の Web プローブを作成します。 プローブが想定どおりに動作するように、次の名前を DNS 内に手動で登録する必要があります。  
  
        1.  directaccess\-directaccess-webprobehost-DirectAccess サーバーの内部 IPv4 アドレス、または IPv6\-のみの環境の IPv6 アドレスに解決する必要があります。  
  
        2.  directaccess\-corpconnectivityhost-localhost \(ループバック\) アドレスに解決する必要があります。 A レコードと AAAA レコードが作成されます。A レコードは 127.0.0.1 の値を持ち、AAAA レコードは NAT64 プレフィックスと最後の 32 ビットが 127.0.0.1 で構成される値を持ちます。 NAT64 プレフィックスを取得するには、コマンドレット get\-get-netnattransitionconfiguration を実行します。  
  
        HTTP または PING 経由で他の Web アドレスを使用して、追加の接続検証方法を作成できます。 接続検証方法ごとに、DNS エントリが存在している必要があります。  
  
#### <a name="dns-server-requirements"></a>DNS サーバーの要件  
  
-   DirectAccess クライアントの場合は、Windows Server 2008、Windows Server 2008 R2、Windows Server 2012、Windows Server 2012 R2、Windows Server 2016、または IPv6 をサポートする任意の DNS サーバーを実行している DNS サーバーを使用する必要があります。  
  
> [!NOTE]  
> DirectAccess をデプロイする場合は、Windows Server 2003 を搭載している DNS サーバーは使用しないことをお勧めします。 Windows Server 2003 の DNS サーバーは IPv6 のレコードをサポートしていますが、Microsoft による Windows Server 2003 のサポートは終了しています。 さらに、ファイル レプリケーション サービスでの問題のため、ドメイン コントローラーが Windows Server 2003 を搭載している場合は、DirectAccess をデプロイしないでください。 詳細については、次を参照してください。 [DirectAccess サポートされない構成](../DirectAccess-Unsupported-Configurations.md)します。  
  
### <a name="bkmk_1_4_NLS"></a>ネットワークロケーションサーバーを計画する  
ネットワーク ロケーション サーバーは、DirectAccess クライアントが企業ネットワーク内に配置されているかどうかを検出するのに使用する Web サイトです。 企業ネットワーク内のクライアントは内部リソースにアクセスするのに DirectAccess を使用せず、リソースに直接接続します。  
  
作業の開始ウィザードでは、ネットワーク ロケーション サーバーが DirectAccess サーバー上に自動的にセットアップされ、DirectAccess の展開時に Web サイトが自動的に作成されます。 これによって、証明書基盤を使用しないシンプルなインストールが実現します。
  
自己\-署名入り証明書を使用せずに、ネットワークロケーションサーバーを展開する場合は、 [「詳細設定を使用して単一の DirectAccess サーバーを展開](../single-server-advanced/Deploy-a-Single-DirectAccess-Server-with-Advanced-Settings.md)する」を参照してください。
  
### <a name="bkmk_1_6_AD"></a>計画 Active Directory  
DirectAccess では、次のように Active Directory および Active Directory グループポリシーオブジェクトを使用します。
  
-   **認証**。 Active Directory を認証に使用します。 DirectAccess トンネルでは、内部リソースにアクセスするユーザーに対して Kerberos 認証を使用します。
  
-   **グループ ポリシー オブジェクト**。 DirectAccess は、DirectAccess サーバーおよびクライアントに適用されるグループ ポリシー オブジェクト内に構成設定を収集します。
  
-   **セキュリティ グループ**。 DirectAccess は、セキュリティ グループを使用して、DirectAccess クライアント コンピューターおよび DirectAccess サーバーを集合化および識別します。 グループ ポリシーは必要なセキュリティ グループに適用されます。

**Active Directory の要件**  
  
DirectAccess 展開用に Active Directory を計画するときは、次のことが必要です。  
  
-   Windows Server 2016、Windows Server 2012 R2、Windows Server 2012、Windows Server 2008 R2、または Windows Server 2008 にインストールされている少なくとも1つのドメインコントローラー。 
  
    ドメインコントローラが境界ネットワーク \(上にあり、そのためにインターネット\-接続している DirectAccess サーバーのネットワークアダプターに接続されている場合\)、インターネットアダプターの IP アドレスへの接続を防ぐために、DirectAccess サーバーがドメインコントローラーにパケットフィルターを追加してアクセスできないようにします。  
  
-   DirectAccess サーバーはドメイン メンバーである必要があります。  
  
-   DirectAccess クライアントは、ドメイン メンバーである必要があります。 クライアントは次に所属することができます。  
  
    -   DirectAccess サーバーと同じフォレスト内の任意のドメイン。  
  
    -   DirectAccess サーバードメインとの2つの\-方向の信頼関係がある任意のドメイン。  
  
    -   DirectAccess ドメインが所属するフォレストと2つの\-方向の信頼関係があるフォレスト内の任意のドメイン。  
  
> [!NOTE]
> - DirectAccess サーバーはドメイン コントローラーになることができません。  
> - DirectAccess に使用される Active Directory ドメインコントローラーは、DirectAccess サーバーの外部インターネットアダプターから到達できないこと \(必要があります。このアダプターは、Windows ファイアウォール\)のドメインプロファイルに含まれていてはなりません。  
  
### <a name="bkmk_1_7_GPOs"></a>グループポリシーオブジェクトの計画  
DirectAccess の構成時に構成された DirectAccess 設定は、GPO\)\(グループポリシーオブジェクトに収集されます。 次に示すとおり、DirectAccess の設定は 2 つの異なる GPO に読み込まれて配布されます。  
  
-   **DirectAccess クライアント GPO**。 この GPO には、IPv6 移行テクノロジ設定、NRPT エントリ、セキュリティが強化された Windows ファイアウォール接続セキュリティの規則を含むクライアント設定が含まれます。 この GPO は、クライアント コンピューターに対して指定されたセキュリティ グループに適用されます。  
  
-   **DirectAccess サーバー GPO**。 この GPO には、デプロイメントの中で DirectAccess サーバーとして構成される任意のサーバーに適用される DirectAccess 構成設定が含まれます。 また、セキュリティが強化された Windows ファイアウォールの接続セキュリティの規則も含まれます。  
  
GPO は 2 つの方法で構成できます。  
  
1.  **自動**。 GPO が自動的に作成されるようにすることを指定できます。 各 GPO には既定の名前が指定されます。 GPO は作業の開始ウィザードによって自動的に作成されます。
  
2.  **手動**。 Active Directory 管理者によって事前に定義されている GPO を使用できます。
  
特定の GPO を使用するように DirectAccess をいったん構成してしまうと、別の GPO の使用を構成できなくなることに注意してください。
  
> [!IMPORTANT]
> 自動または手動で構成された GPO のどちらを使用するにせよ、クライアントが 3G を使用する場合は、低速リンクの検出用のポリシーを追加する必要があります。 ポリシーのグループポリシーパス **: 構成グループポリシー低速リンクの検出**は、**コンピューターの構成 \/ ポリシー \/ 管理用テンプレート \/ システム \/ グループポリシー**です。  
  
> [!CAUTION]  
> Directaccess コマンドレットを実行する前にすべての DirectAccess グループポリシーオブジェクトをバックアップするには、次の手順を使用します。 directaccess[構成のバックアップと復元](https://go.microsoft.com/fwlink/?LinkID=257928)  
  
#### <a name="automatically-created-gpos"></a>作成した Gpo を自動的に\-する  
自動\-作成された Gpo を使用する場合は、次の点に注意してください。  
  
自動的に作成された GPO は、次に示すように、場所とリンク先のパラメーターに基づいて適用されます。  
  
-   DirectAccess サーバー GPO の場合、場所とリンクのパラメーターはどちらも、DirectAccess サーバーを含むドメインを示します。  
  
-   クライアント GPO が作成されると、場所は GPO の作成先となる単一ドメインに設定されます。 GPO 名は各ドメインで検索され、存在する場合は DirectAccess の設定が読み込まれます。 リンク先は GPO が作成されたドメインのルートに設定されます。 GPO はクライアント コンピューターを含むドメインごとに作成され、その個々のドメインのルートにリンクされます。  
  
自動で作成された GPO を使用しているときに DirectAccess の設定を適用する場合、DirectAccess サーバー管理者は次のアクセス許可が必要になります。  
  
-   各ドメインの GPO 作成アクセス許可。  
  
-   選択したすべてのクライアント ドメイン ルートのリンク アクセス許可。  
  
-   サーバー GPO ドメイン ルートのリンク アクセス許可。  
  
-   セキュリティの作成、編集、削除、および変更のアクセス許可が GPO には必要です。  
  
-   DirectAccess 管理者が、必要なドメインごとに GPO 読み取りアクセス許可を持つことが推奨されます。 これによって、GPO の作成時に、重複する名前の GPO が存在しないことを DirectAccess が確認できるようになります。  
  
GPO をリンクするための正しいアクセス許可がない場合は、警告が表示されることに注意してください。 DirectAccess の操作は続行されますが、リンク処理は行われません。 この警告が表示された場合は、後でアクセス許可を追加しても、リンクは自動的には作成されません。 代わりに、管理者がリンクを手動で作成する必要があります。  
  
#### <a name="manually-created-gpos"></a>手動\-作成された Gpo  
手動\-作成した Gpo を使用する場合は、次の点に注意してください。  
  
-   GPO はリモート アクセスの作業の開始ウィザードを実行する前に存在している必要があります。  
  
-   手動\-作成した Gpo を使用する場合は、DirectAccess の設定を適用するために、DirectAccess 管理者が gpo を手動\-作成したときに、完全な GPO のアクセス許可 \(編集、削除、\) 変更を行う必要があります。  
  
-   手動で作成された GPO を使用しているときは、GPO へのリンクがドメイン全体で検索されます。 GPO がドメイン内でリンクされていない場合は、ドメイン ルート内にリンクが自動的に作成されます。 リンクを作成するために必要なアクセス許可がない場合は、警告が表示されます。  
  
GPO をリンクするための正しいアクセス許可がない場合は、警告が表示されることに注意してください。 DirectAccess の操作は続行されますが、リンク処理は行われません。 この警告が表示された場合は、後でアクセス許可を追加しても、リンクは自動的には作成されません。 代わりに、管理者がリンクを手動で作成する必要があります。  
  
#### <a name="recovering-from-a-deleted-gpo"></a>削除された GPO からの回復  
DirectAccess サーバー、クライアント、またはアプリケーションサーバーの GPO が誤って削除され、使用できるバックアップがない場合は、構成設定を削除して、再度構成\-必要があります。 バックアップを使用できる場合は、バックアップから GPO を復元できます。  
  
**DirectAccess の管理**で、 **GPO <GPO name> が見つからないこと**を示すエラーメッセージが表示されます。 構成設定を削除するには、次の手順に従います。  
  
1.  PowerShell コマンドレットを実行して、 **\-remoteaccess をアンインストール**します。  
  
2.  **DirectAccess 管理**を\-再起動します。  
  
3.  GPO が見つからないというエラー メッセージが表示されます。 **[構成設定の削除]** をクリックします。 完了後、サーバーは\-構成されていない状態に復元されます。  
  
### <a name="BKMK_Links"></a>次のステップ  
  
-   [手順 2: 基本的な DirectAccess 展開を計画する](da-basic-plan-s2-deployment.md)  
  
