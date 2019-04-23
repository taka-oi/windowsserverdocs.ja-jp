---
ms.assetid: c28c60ff-693d-49ee-a75b-58f24866217b
title: "フェデレーション サーバー プロキシの名前解決の要件"
description: 
author: billmath
ms.author: billmath
manager: femila
ms.date: 05/31/2017
ms.topic: article
ms.prod: windows-server-threshold
ms.technology: identity-adfs
ms.openlocfilehash: a94e4de181cd8794d479bbd6695a94658aba0f86
ms.sourcegitcommit: db290fa07e9d50686667bfba3969e20377548504
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/12/2017
---
# <a name="name-resolution-requirements-for-federation-server-proxies"></a>フェデレーション サーバー プロキシの名前解決の要件

>適用対象: Windows Server 2016、Windows Server 2012 R2、Windows Server 2012

インターネット上のクライアント コンピューターが Active Directory フェデレーション サービス \(AD FS\) で保護されているアプリケーションにアクセスしようとすると、最初のフェデレーション サーバーを認証する必要があります。 ほとんどの場合、フェデレーション サーバーは通常ありません、インターネットから直接アクセスです。 そのため、インターネット クライアント コンピューターがリダイレクトされるフェデレーション サーバー プロキシを代わりにします。 またはインターネットが直面する複数の DNS ゾーンに適切なドメイン ネーム システム \(DNS\) レコードを追加することで、正常なリダイレクトを行うことができます。  
  
フェデレーション サーバー プロキシをインターネット クライアントをリダイレクトするために使用する方法は、境界ネットワーク内の DNS ゾーンを構成する方法や、インターネット上を制御する DNS ゾーンを構成する方法によって異なります。 フェデレーション サーバー プロキシが境界ネットワークで使用するものです。 インターネット クライアント要求をリダイレクト フェデレーション サーバーに正常に制御するすべてのとは限らないに接続されたゾーンで DNS が正しく構成されている場合にのみ。 限らないに接続されたゾーンの構成ではそのため、:、境界ネットワークのみを提供している DNS ゾーンまたは境界ネットワークとインターネット クライアントの両方を提供している DNS ゾーンがあるかどうか-が重要です。  
  
このトピックでは、フェデレーション サーバー プロキシを境界ネットワーク内に配置すると、名前解決を構成する手順について説明します。 使用する手順を調べるには、まず、次の DNS シナリオのどれに最も近い、組織の境界ネットワーク内の DNS インフラストラクチャを確認します。 次に、そのシナリオの手順に従ってください。  
  
## <a name="dns-zone-serving-only-the-perimeter-network"></a>境界ネットワークのみを提供している DNS ゾーン  
このシナリオで、組織が、境界ネットワーク内の 1 つまたは 2 つの DNS ゾーンにあり、組織では、インターネット上のすべての DNS ゾーンを制御しません。 境界ネットワークのシナリオのみを提供する DNS ゾーンでフェデレーション サーバー プロキシの正常な名前の解決策は、次の条件によって異なります。  
  
-   フェデレーション サーバー プロキシの設定が必要に hosts ファイルに解決するには、完全修飾ドメイン名をフェデレーション サーバーまたはフェデレーション サーバー クラスターの IP アドレスにフェデレーション サーバーのエンドポイント URL の \(FQDN\) します。  
  
-   フェデレーション サーバーのエンドポイント URL の FQDN は、フェデレーション サーバー プロキシの IP アドレスに解決できるように、アカウント パートナーの境界ネットワークに DNS を構成する必要があります。  
  
次の図と対応する手順は、これらの各条件を特定の例が実現する方法を示しています。 この図では、Microsoft Network Load Balancing \(NLB\) テクノロジは、既存のフェデレーション サーバー ファームの 1 つ、クラスターの FQDN と 1 つ、クラスター IP アドレスを提供します。  
  
![名の要件](media/adfs2_deploy_single_fs.gif)  
  
クラスターの IP アドレスまたはクラスター FQDN の構成の詳細については、NLB を使用してを参照してください[クラスター パラメーターを指定して](https://go.microsoft.com/fwlink/?LinkId=75282)します。  
  
### <a name="1-configure-the-hosts-file-on-the-federation-server-proxy"></a>1。フェデレーション サーバー プロキシで hosts ファイルを構成します。  
アカウント パートナーのフェデレーション サーバー プロキシが fs.fabrikam.com を実際のアカウント フェデレーション サーバーの IP アドレスに解決するのには、ローカルの hosts ファイルにエントリを持つアカウント フェデレーション サーバー プロキシを fs.fabrikam.com に対するすべての要求を解決するのには、境界ネットワークに DNS が構成されている、ため \ (またはフェデレーション サーバー farm\ のクラスター DNS 名) は、企業ネットワークに接続されています。 これにより、それ自体にではなく、アカウント フェデレーション サーバーに fs.fabrikam.com ホスト名を解決するのにはアカウント フェデレーション サーバー プロキシの-境界 DNS を使用して fs.fabrikam.com を検索した場合に発生すると: フェデレーション サーバー プロキシは、フェデレーション サーバーと通信できるようにします。  
  
### <a name="2-configure-perimeter-dns"></a>2. 境界 DNS を構成します。  
のみ単一 AD FS のホスト名をクライアント コンピューターが送信されるため、イントラネットまたはインターネット上であるかどうか-境界 DNS サーバーを使用して、インターネット上のクライアント コンピューターは、境界ネットワーク上のアカウント フェデレーション サーバー プロキシの IP アドレスにアカウント フェデレーション サーバー \(fs.fabrikam.com\) の FQDN を解決する必要があります。 fs.fabrikam.com の解決を試みたときに、ログオン アカウント フェデレーション サーバー プロキシのクライアントを転送できる、できるように、境界 DNS には、fs \(fs.fabrikam.com\) の 1 つのホスト \(A\) リソース レコードと、境界ネットワーク上のアカウント フェデレーション サーバー プロキシの IP アドレスの限られた corp.fabrikam.com DNS ゾーンが含まれています。  
  
フェデレーション サーバー プロキシの hosts ファイルを変更して、境界ネットワークに DNS を構成する方法の詳細については、次を参照してください。[、境界ネットワークのみを提供する DNS ゾーンでフェデレーション サーバー プロキシの名前解決を構成する](../../ad-fs/deployment/Configure-Name-Resolution-for-a-Federation-Server-Proxy-in-a-DNS-Zone-That-Serves-Only-the-Perimeter-Network.md)します。  
  
## <a name="dns-zone-serving-both-the-perimeter-network-and-internet-clients"></a>境界ネットワークとインターネット クライアントの両方を提供している DNS ゾーン  
このシナリオでは、組織は、境界ネットワーク内の DNS ゾーンと、インターネット上に少なくとも 1 つの DNS ゾーンを制御します。 このシナリオでフェデレーション サーバー プロキシの正常な名前の解決策は、次の条件によって異なります。  
  
-   フェデレーション サーバーのホスト名の FQDN が境界ネットワーク内のフェデレーション サーバー プロキシの IP アドレスに解決できるように、アカウント パートナーのインターネット ゾーンに DNS を構成する必要があります。  
  
-   フェデレーション サーバーのホスト名の FQDN が企業ネットワーク内のフェデレーション サーバーの IP アドレスに解決できるように、アカウント パートナーの境界ネットワークに DNS を構成する必要があります。  
  
次の図と対応する手順は、これらの各条件を特定の例が実現する方法を示しています。  
  
![名の要件](media/adfs2_deploy_fsp_3DNS.gif)  
  
### <a name="1-configure-perimeter-dns"></a>1。境界 DNS を構成します。  
このシナリオでは、インターネット DNS ゾーンを構成することと見なされますので解決するのには制御する要求が行われる特定のエンドポイント URL の \ (つまり、fs.fabrikam.com\)、境界ネットワーク内のフェデレーション サーバー プロキシ、境界の企業ネットワーク内のフェデレーション サーバーにこれらの要求を転送するように DNS のゾーンも構成する必要があります。  
  
ように fs.fabrikam.com の解決を試みたときに、クライアントをアカウント フェデレーション サーバーに転送できる、境界 DNS で構成されている fs \(fs.fabrikam.com\) と企業ネットワーク上のアカウント フェデレーション サーバーの IP アドレスの 1 つのホスト \(A\) リソース レコードです。 これにより、それ自体にではなく、アカウント フェデレーション サーバーに fs.fabrikam.com ホスト名を解決するのにはアカウント フェデレーション サーバー プロキシの-インターネット DNS を使用して fs.fabrikam.com を検索した場合に発生すると: フェデレーション サーバー プロキシは、フェデレーション サーバーと通信できるようにします。  
  
### <a name="2-configure-internet-dns"></a>2. インターネット DNS を構成します。  
このシナリオでは正常に行われるには、名前の解決の fs.fabrikam.com、インターネット上のクライアント コンピューターからすべての要求で解決しなければならないインターネット DNS ゾーンを制御できること。 そのため、境界ネットワーク内のアカウント フェデレーション サーバー プロキシの IP アドレスを fs.fabrikam.com に対するクライアント要求を転送するように、インターネット DNS ゾーンを構成する必要があります。  
  
境界ネットワークとインターネットの DNS ゾーンを変更する方法の詳細については、次を参照してください。[DNS ゾーンことはどちらも、境界ネットワークとインターネット クライアントでのフェデレーション サーバー プロキシの名前解決を構成する](../../ad-fs/deployment/Configure-Name-Resolution-for-a-Federation-Server-Proxy-in-a-DNS-Zone-That-Serves-Both-the-Perimeter-Network-and-Internet-Clients.md)します。  
  
## <a name="see-also"></a>参照してください。
[Windows Server 2012 で AD FS 設計ガイドします。](AD-FS-Design-Guide-in-Windows-Server-2012.md)