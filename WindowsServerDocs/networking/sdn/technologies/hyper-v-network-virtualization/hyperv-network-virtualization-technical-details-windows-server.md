---
title: Windows Server 2016 で HYPER-V ネットワーク仮想化の技術的な詳細
description: このトピックの「Windows Server 2016 で HYPER-V ネットワーク仮想化に関する技術情報を提供します。
manager: brianlic
ms.custom: na
ms.prod: windows-server-threshold
ms.reviewer: na
ms.service: virtual-network
ms.suite: na
ms.technology:
- networking-sdn
ms.tgt_pltfrm: na
ms.topic: article
ms.assetid: 9efe0231-94c1-4de7-be8e-becc2af84e69
ms.author: pashort
author: shortpatti
ms.openlocfilehash: af2b2a0b151601124bb473c465e7d5a97f2b9150
ms.sourcegitcommit: 19d9da87d87c9eefbca7a3443d2b1df486b0b010
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/28/2018
---
# <a name="hyper-v-network-virtualization-technical-details-in-windows-server-2016"></a>Windows Server 2016 で HYPER-V ネットワーク仮想化の技術的な詳細

>Windows Server 2016 の適用対象:

サーバー仮想化では、1 つの物理ホストで同時に実行する複数のサーバーまだサーバー インスタンスは互いから分離されます。 各仮想マシンでは、物理コンピューターで実行されている唯一のサーバーである場合、本質的に動作します。  
  
どの (可能性のあるで重複する IP アドレス) の複数の仮想ネットワークが同じ実行で物理ネットワーク インフラストラクチャと各仮想ネットワークように動作は、共有ネットワーク インフラストラクチャで実行されている唯一の仮想ネットワーク、ネットワーク仮想化は、同様の機能を提供します。 図 1 は、この関係を示します。  
  
![サーバー仮想化とネットワーク仮想化](../../../media/hyper-v-network-virtualization-technical-details-in-windows-server/VNetF1.gif)  
  
図 1: サーバー仮想化は、ネットワーク仮想化と  
  
## <a name="hyper-v-network-virtualization-concepts"></a>HYPER-V ネットワーク仮想化の概念  
HYPER-V ネットワーク仮想化 (HNV) では、お客様またはテナントは、企業やデータ センターに展開されている IP サブネットのセットの「所有者」として定義されます。 顧客には、企業または企業で複数の部門やネットワークの分離を必要とするプライベート データ センター内の事業単位やサービス プロバイダーによってホストされているパブリック データ センター内のテナントを指定できます。 各顧客は、1 つまたは複数を所有でき[仮想ネットワーク](#VirtualNetworks)、データ センター内およびそれぞれの仮想ネットワークは、1 つまたは複数[仮想サブネット](#VirtualSubnets)します。  
  
Windows Server 2016 で利用可能になりますが、2 つの HNV 実装がある: HNVv1 と HNVv2 します。  
  
-   **HNVv1**  
  
    HNVv1 は Windows Server 2012 R2 および System Center 2012 R2 Virtual Machine Manager (VMM) との互換性です。 HNVv1 の構成では、物理アドレス (PA) マッピングとルーティングに分離の設定とカスタマー アドレス (CA) の仮想ネットワークを定義する WMI 管理と Windows PowerShell コマンドレット (System Center VMM を容易になります) に依存します。 Windows Server 2016 で HNVv1 する追加機能が追加されていないの新機能が予定されていません。  
    
    • セットのチーム化と HNV V1 はプラットフォームによって互換性がありません。
    
    O HA NVGRE ゲートウェイのユーザーを使用する必要がありますいずれかを使用して LBFO チームまたはチームなし。 または
    
    O セットでゲートウェイを使用してネットワーク コントローラーの展開には、スイッチがチーム化されました。

  
-   **HNVv2**  
  
    多数の新しい機能は、Azure 仮想フィルタ リング プラットフォーム (VFP) 転送 - HYPER-V スイッチ拡張機能を使用して実装されている HNVv2 で含まれています。 HNVv2 完全に統合されて、ソフトウェアによるネットワーク制御 (SDN) スタックで、新しいネットワーク コント ローラーを含む Microsoft Azure Stack します。  仮想ネットワーク ポリシーが、Microsoft によって定義されている[ネットワーク コント ローラー](../../../sdn/technologies/network-controller/Network-Controller.md) 、RESTful NorthBound (NB) API を使用して、ホストのエージェント経由で複数 SouthBound インターフェイス (SBI) OVSDB を含むに組み込まします。 ホストのエージェントのポリシーが強制される HYPER-V スイッチの VFP 拡張機能にプログラムします。  
  
    > [!IMPORTANT]  
    > このトピックでは、HNVv2 について説明します。  
  
### <a name="VirtualNetworks"></a>仮想ネットワーク  
  
-   各仮想ネットワークは、1 つまたは複数の仮想サブネットで構成されます。 仮想ネットワークは、仮想ネットワーク内の仮想マシンが相互に通信のみここで分離境界を形成します。 従来は、このような分離が強制適用された分離された IP アドレスの範囲と 802.1q Vlan を使用してタグや VLAN の id。 お客様またはテナントの間で IP サブネットを重複する可能性はありますが、オーバーレイ ネットワークを作成するか、NVGRE または VXLAN カプセル化を使用して分離を強制、HNV で。  
  
-   各仮想ネットワークには、ホスト上、一意のルーティング ドメイン ID (RDID) があります。 この RDID は約仮想ネットワーク、ネットワーク コント ローラーの REST リソースを識別するリソース ID にマップされます。 Uniform Resource Identifier (URI) 名前空間を使用して、追加のリソース ID を持つ仮想ネットワークの他のリソースが参照されます。  
  
### <a name="VirtualSubnets"></a>仮想サブネット  
  
-   仮想サブネットでは、同じ仮想サブネット内のバーチャル マシンに対して、レイヤー 3 IP サブネット セマンティクスを実装します。 仮想サブネットはブロードキャスト ドメイン (VLAN に似ています) を形成し、分離は、NVGRE テナント ネットワーク ID (TNI) または VXLAN ネットワーク識別子 (VNI) のいずれかのフィールドを使用して適用します。  
  
-   各仮想サブネットに単一の仮想ネットワーク (RDID) は、属し、一意の仮想サブネット ID (VSID) カプセル化されたパケット ヘッダーの TNI または VNI のいずれかのキーを使用してが割り当てられています。 VSID はデータ センター内で一意である必要があり、4096 ~ 2 の範囲内にある ^24-2 です。  
  
仮想ネットワークとルーティング ドメインの主な利点は、できるお客様は (たとえば、IP サブネット)、独自のネットワーク トポロジをクラウドにします。 図 2 では、Contoso Corp は R & D Net と Sales Net 2 つの独立したネットワークいます例を示します。 これらのネットワークに別のルーティング ドメイン Id があるために、相互に合うことはできません。 つまり、Contoso R & D Net は Contoso Sales Net から切り離さ場合でも、両方が所有している Contoso 社。 Contoso R & D Net 3 つの仮想サブネットが含まれています。 RDID と VSID の両方が、データ センター内で一意であるに注意してください。  
  
![カスタマー ネットワークと仮想サブネット](../../../media/hyper-v-network-virtualization-technical-details-in-windows-server/VNetF6.gif)  
  
図 2: カスタマー ネットワークと仮想サブネット  
  
**レイヤー 2 の転送**  
  
図 2 は、VSID 5001 の仮想マシンも、HYPER-V スイッチを通過 VSID 5001 の仮想マシンに転送されたパケットはそのことができます。 VSID 5001 の仮想マシンからの受信パケットは、HYPER-V スイッチで特定 VPort に送信されます。 受信の規則 (例: 全て) およびマッピング (カプセル化ヘッダーなど) は、これらのパケットを HYPER-V スイッチによって適用されます。 パケットが、転送、HYPER-V スイッチで異なる VPort のいずれか (送信先仮想マシンは、同じホストに接続されている) 場合または別のホスト (送信先仮想マシンが別のホスト上にある) 場合に別の HYPER-V スイッチにします。  
  
**レイヤー 3 のルーティング**  
  
同様に、VSID 5001 の仮想マシン パケットを VSID 5002 または VSID 5003 の仮想マシンにルーティングされます HNV 分散ルーターは、これは各 HYPER-V ホストの VSwitch に存在することができます。 パケットを HYPER-V に提供時に切り替えると、HNV、着信パケットの VSID を送信先仮想マシンの VSID に更新します。 これは両方の Vsid が同じ RDID 内場合のみ発生します。  したがって、rdid1 の仮想ネットワーク アダプターはゲートウェイを通過することがなく rdid2 の仮想ネットワーク アダプターにパケットを送信ことはできません。  
  
> [!NOTE]  
> パケット フロー上の説明、という用語"virtual machine"実際には、仮想マシン上の仮想ネットワーク アダプターです。 一般的なケースは、仮想マシンが単一の仮想ネットワーク アダプターのみがあります。 この場合は、「仮想マシン」と「仮想ネットワーク アダプタ」できます概念的には同じことを意味します。  
  
各仮想サブネットでは、レイヤー 3 IP サブネットおよび VLAN に似たレイヤー 2 (L2) ブロードキャスト ドメイン境界を定義します。 ときに、仮想マシンがパケットをブロードキャスト、HNV は、元のパケットのコピーを作成し、同じ VSID に含まれている各 VM のアドレスで移行先の IP と MAC を置き換えるユニキャスト レプリケーション (UR) を使用します。  
  
> [!NOTE]  
> Windows Server 2016 が付属している場合、ユニキャスト レプリケーションを使用してブロードキャストとサブネットのマルチキャストが実装されます。 サブネット間でのマルチキャスト ルーティングおよび IGMP はサポートされていません。  
  
VSID はブロードキャスト ドメインであるだけでなく、分離を提供します。 HNV の仮想ネットワーク アダプターは、ポート (virtualNetworkInterface REST リソース) に直接または一部が仮想サブネット (VSID) に適用する ACL の規則を持つ HYPER-V スイッチ ポートに接続されます。  
  
HYPER-V スイッチ ポート ACL のルールを適用が必要です。 この ACL はすべて許可、すべて拒否するまたは詳細にのみ特定の種類の 5 組 (発信元 IP、宛先 IP、発信元ポート、宛先ポート、プロトコル) に一致するに基づいて、トラフィックを許可する指定できませんでした。  
  
> [!NOTE]  
> HYPER-V スイッチ拡張機能は、新しいソフトウェアによるネットワーク制御 (SDN) スタックで HNVv2 で機能しません。 HNVv2 は、その他のサード パーティ製スイッチ拡張機能と組み合わせて使用することはできません Azure 仮想フィルタ リング プラットフォーム (VFP) のスイッチ拡張機能を使用して実装されます。  
  
## <a name="switching-and-routing-in-hyper-v-network-virtualization"></a>切り替えと HYPER-V ネットワーク仮想化のルーティング  
HNVv2 実装正しいレイヤー 2 (L2) を切り替えると、物理スイッチと同様に動作するレイヤー 3 (L3) ルーティング セマンティクスまたはルーターが機能します。 仮想マシンに接続 HNV の仮想ネットワークは、同じ仮想サブネット (VSID) は最初にリモートの仮想マシンの CA の MAC アドレスを学習する必要があります内で別の仮想マシン接続を確立しようとします。 ソース バーチャル マシンの ARP の表に、移行先の仮想マシンの IP アドレスの ARP エントリがある場合は、このエントリから、MAC アドレスが使用されます。 エントリが存在しない場合、ソース バーチャル マシンは、ARP が返される送信先仮想マシンの IP アドレスに対応する MAC アドレスの要求とブロードキャストを送信します。 HYPER-V スイッチはこの要求を受け取るし、ホストのエージェントに送信します。 ホストのエージェントは、要求された移行先の仮想マシンの IP アドレスに対応する MAC アドレスをローカル データベースになります。  
  
> [!NOTE]  
> OVSDB サーバーとして機能する、ホストのエージェントは、CA と PA のマッピング、MAC の表を格納するのに VTEP スキーマのバリアントを使用します。  
  
MAC アドレスが利用可能な場合は、ホストのエージェントは、ARP 応答が挿入し、仮想マシンに送信します。 バーチャル マシンのネットワーク スタックは、すべての必要な L2 ヘッダー情報がある、フレームが V スイッチの対応する HYPER-V ポートに送信されます。 内部的には、HYPER-V スイッチでは、このフレームに対して V ポートに割り当てられている N 組一致ルールをテストし、これらのルールに基づいて、フレームに特定の変換を適用します。 最も重要なは、NVGRE または VXLAN のいずれかを使用して、ネットワーク コント ローラーで定義されているポリシーによってカプセル化ヘッダーを構築する一連のカプセル化変換が適用されます。 ホストのエージェントでプログラミングされているポリシーに基づき、CA と PA のマッピングは使用、HYPER-V ホストの IP アドレスを決定する送信先仮想マシンが存在します。 HYPER-V スイッチにより、適切なルーティングの規則と、リモートの PA アドレスに到達するために VLAN タグが外側のパケットに適用されます。  
  
HNV の仮想ネットワークに接続されている仮想マシンは、別の仮想サブネット (VSID) 内の仮想マシンの接続を作成する必要がある場合、パケットを適切にルーティングする必要があります。 HNV は、星トポロジを想定しています。 1 つだけの IP アドレス、next-hop としてすべての IP プレフィックス (意味 1 つの既定のルート/ゲートウェイ) に到達するために使用する CA 空間が存在します。 現時点では、これにより、1 つの既定のルートに制限を強制し、既定以外のルートがサポートされていません。  
  
### <a name="routing-between-virtual-subnets"></a>仮想サブネット間のルーティング  
物理ネットワークでは、IP サブネットは、ここでコンピュータ (仮想および物理) は、相互に通信できる直接レイヤー 2 (L2) ドメインです。 L2 ドメインは、すべてのインターフェイスにブロードキャストされる ARP 要求を通じて ARP エントリ (IP:MAC アドレス マップ) を学習、ARP 応答は要求元のホストに送信されること、ブロードキャスト ドメインです。 コンピューターでは、ARP 応答から学習した MAC 情報を使用して、イーサネット ヘッダーを含む、L2 フレームを完全に作成します。 ただし、IP アドレスが異なる L3 サブネット内にある場合は、ARP 要求はこの L3 境界を通過しません。 代わりに、ソース サブネットの IP アドレスを持つ L3 ルーター インターフェイス (次ホップまたはデフォルト ゲートウェイ) は、独自の MAC アドレスを持つ場合は、これら ARP 要求に応答する必要があります。  
  
標準の Windows ネットワークでは、管理者は静的ルートを作成し、ネットワーク インターフェイスに割り当てます。 さらに、「既定のゲートウェイ」は通常、既定のルート (0.0.0.0/0) に送信されるパケットが送信されるインターフェイスで、次ホップ IP アドレスに構成します。 パケットは、特定のルートが存在しない場合、この既定のゲートウェイに送信されます。 これは、物理ネットワークのルーターでは通常です。  HNV は、すべてのホストの一部であるでの仮想ネットワークの分散ルーターを作成するには、各 VSID インターフェイスを備えた組み込みルーターを使用します。  
  
HNV は、星のトポロジを想定しています、ために、HNV の分散ルーターは同じ VSID ネットワークの一部である仮想サブネット間で移動するすべてのトラフィックの 1 つのデフォルト ゲートウェイとして機能します。 アドレスは、デフォルト ゲートウェイ既定値は、VSID の最小の IP アドレスとして使用され、HNV の分散ルーターに割り当てられています。 この分散ルーターは、各ホストは仲介を必要とせず、適切なホストにトラフィックを直接ルーティングできるので、適切にルーティングする VSID ネットワーク内のすべてのトラフィックの非常に効率的な方法になります。  これは、同じ VM ネットワークが別の仮想サブネット内の 2 つの仮想マシンが同じ物理ホスト上にある場合に特に当てはまります。  わかりますが後でこのセクションで、パケットは、物理ホストをそのまま使用することはありませんがあります。  
  
### <a name="routing-between-pa-subnets"></a>PA サブネット間のルーティング  
対照的に 1 つの PA IP アドレスの各仮想サブネット (VSID) を割り当て HNVv1、HNVv2 は今すぐ Switch-Embedded チーミング (SET) の NIC チームのメンバーごとに 1 つの PA IP アドレスを使用します。 既定の展開では、2 枚の NIC チームを前提としていて、ホストごとに 2 つの PA IP アドレスを割り当てます。 1 つのホストには、同じ VLAN で同じプロバイダー (PA) の論理サブネットから割り当てられている PA ip アドレスがあります。 同じ仮想サブネット内の 2 つのテナント Vm は、2 つの別のプロバイダー論理サブネットに接続されている 2 つの異なるホスト上で実際に存在します。 HNV では、CA と PA のマッピングに基づくカプセル化されたパケットの場合、外部の IP ヘッダーを構築します。 ただし、既定の PA ゲートウェイの ARP にホストの TCP/IP スタックが依存し、ARP 応答に基づいて、外部イーサネット ヘッダーを作成します。 通常、この ARP 応答は SVI インターフェイスでの物理スイッチまたは L3 ルーター、ホストが接続されています。 HNV はそのためプロバイダー論理サブネット間でカプセル化されたパケットをルーティングする L3 ルーターに依存/Vlan します。  
  
### <a name="routing-outside-a-virtual-network"></a>仮想ネットワークの外側のルーティング  
ほとんどの顧客展開には、HNV 環境から HNV 環境の一部ではないリソースへの通信が必要です。 ネットワーク仮想化ゲートウェイは、2 つの環境間の通信を許可する必要があります。 プライベート クラウドやハイブリッド クラウド インフラストラクチャには、HNV ゲートウェイを必要とするが含まれます。 基本的には、HNV ゲートウェイは、レイヤー 3 のルーティング内部および外部の (物理) ネットワークを使用して、(NAT を含む) の間、または別のサイトや、IPSec VPN または GRE トンネルを使用するクラウド (プライベートまたはパブリック) の間で必要です。  
  
ゲートウェイは、さまざまな物理フォーム ファクターで提供できます。 これらは、トップ オブ ラック (TOR) スイッチとして動作しているアクセスによって、仮想 IP (VIP)、ロード バランサーでアドバタイズ、VXLAN ゲートウェイの他の既存のネットワーク アプライアンスにまたは新しいスタンドアロンのネットワーク アプライアンスにすることができますに組み込まれている、Windows Server 2016 上で構築できます。  
  
Windows RAS ゲートウェイのオプションの詳細については、次を参照してください。 [RAS ゲートウェイ](../../../../remote/remote-access/ras-gateway/RAS-Gateway.md)します。  
  
## <a name="packet-encapsulation"></a>パケット カプセル化  
HNV の各仮想ネットワーク アダプターが 2 つの IP アドレスに関連付けられます。  
  
-   **カスタマー アドレス**(CA) の IP アドレスがそのイントラネット インフラストラクチャに基づいて、顧客が割り当てられています。 このアドレスをパブリックまたはプライベート クラウドに移行された意識せずに、バーチャル マシンでのネットワーク トラフィックをやり取りすることができます。 CA は、仮想マシンから見える、顧客が到達可能なです。  
  
-   **プロバイダー アドレス**、ホスティング プロバイダーによって割り当てられた (PA) IP アドレスまたはその物理ネットワーク インフラストラクチャに基づいてデータ センターの管理者。 PA は、仮想マシンをホストしている HYPER-V を実行しているサーバーの間で交換されるネットワーク上のパケットに表示されます。 PA が表示される仮想マシンではなく、物理ネットワークでします。  
  
Ca は、仮想化され、実際の基礎となる物理ネットワーク トポロジとアドレスから切り離すことが、PAs に実装されたお客様のネットワーク トポロジを維持します。 次の図は、仮想マシンの Ca とネットワーク仮想化の結果としてネットワーク インフラストラクチャの Pa の概念上の関係を示します。  
  
![物理インフラストラクチャ上のネットワーク仮想化の概念図](../../../media/hyper-v-network-virtualization-technical-details-in-windows-server/VNetF2.gif)  
  
物理インフラストラクチャ上のネットワーク仮想化の概念図を図 6:  
  
図では、ユーザーの仮想マシンはデータ パケットを送信 CA 空間内を通じて、独自の仮想ネットワーク、または「トンネル」物理ネットワーク インフラストラクチャを横断します。 上記の例では、トンネルできますと見なす Contoso と Fabrikam のデータ パケットと緑の宛先ラベル (PA アドレス)、左側の送信元ホストから右側の送信先ホストに配信される包む「封筒」します。 キーは、ホストが「宛先アドレス」を確認する方法 (PA) が、Contoso と Fabrikam の CA、パケットに「封筒」に包まれ方法およびし方法は、送信先ホスト パケットを開封して Contoso と Fabrikam の送信先仮想マシンに正しく送り届けるに対応します。  
  
この単純な例では、ネットワーク仮想化の主な側面が強調表示されます。  
  
-   各仮想マシンの CA は物理ホスト PA. にマップされます。 同じ PA. に関連付けられている複数の Ca があります。  
  
-   仮想マシンは、CA は、空白と一緒に「封筒」PA の送信元と送信先のペアを使用して、マッピングに基づいてに入れられますデータ パケットを送信します。  
  
-   CA と PA のマッピングには、さまざまなカスタマー仮想マシンのパケットを区別するためにホストができるようにする必要があります。  
  
その結果、ネットワーク仮想化メカニズムは、仮想マシンで使用されるネットワーク アドレスを仮想化です。 ネットワーク コント ローラーは、アドレスのマッピングを担当し、ホストのエージェントがマッピングを MS_VTEP スキーマを使用してデータベースを維持します。 次のセクションでは、実際のアドレス仮想化メカニズムについて説明します。  
  
## <a name="network-virtualization-through-address-virtualization"></a>アドレス仮想化によるネットワーク仮想化  
オーバーレイ ネットワーク仮想化 Generic Routing Encapsulation (NVGRE) または仮想拡張可能なローカル エリア ネットワーク (VXLAN) のいずれかを使用して、テナント ネットワークの HNV 実装します。  VXLAN 既定値です。  
  
### <a name="virtual-extensible-local-area-network-vxlan"></a>仮想拡張可能なローカル エリア ネットワーク (VXLAN)  
仮想拡張可能なローカル エリア ネットワーク (VXLAN) ([RFC 7348](http://www.rfc-editor.org/info/rfc7348)) プロトコルが Cisco、Brocade、Arista、Dell、HP および他のユーザーのようなベンダーからのサポートを市場で広く受け入れられます。 VXLAN プロトコルは、UDP をトランスポートとして使用します。 VXLAN の IANA 割り当ての UDP 宛先ポートは 4789、UDP 発信元ポートは、ECMP を展開するための内側のパケットからの情報のハッシュをする必要があります。 UDP ヘッダーの後は、VXLAN ヘッダーは 3 バイトのフィールドに、VXLAN ネットワーク識別子 (VNI) - VSID の予約済みの別の 1 バイト フィールド続けて後に予約されている 4 バイト フィールドが含まれているパケットに追加されます。 VXLAN ヘッダーの後、元の CA L2 フレーム (なし、CA イーサネット フレーム FCS) が追加されます。  
  
![VXLAN パケット ヘッダー](../../../media/hyper-v-network-virtualization-technical-details-in-windows-server/VXLAN-packet-header.png)  
  
### <a name="generic-routing-encapsulation-nvgre"></a>汎用ルーティング カプセル化 (NVGRE)  
このネットワーク仮想化メカニズムがトンネル ヘッダーの一部として、Generic Routing Encapsulation (NVGRE) を使用します。 Nvgre では、仮想マシンのパケットは別のパケット内カプセル化されます。 この新しいパケットのヘッダーには適切なレプリケーション元と送信先の PA IP アドレス、図 7 に示すように、GRE ヘッダーのキー フィールドに格納されている仮想サブネット ID に加えてします。  
  
![NVGRE カプセル化](../../../media/hyper-v-network-virtualization-technical-details-in-windows-server/VNetF3.gif)  
  
図 7: ネットワーク仮想化 - NVGRE カプセル化  
  
仮想サブネット ID は、ホストのすべてのパケットについてカスタマー仮想マシンを識別できるように、パケット上の PA と CA の場合でも重複する可能性があります。 これにより、すべてのバーチャル マシンを単一の PA を共有する同じホスト上で図 7 に示すようにします。  
  
PA の共有は、ネットワークのスケーラビリティに大きな影響があります。 IP アドレスと MAC アドレス、ネットワーク インフラストラクチャに覚えさせる必要があるの数を大幅に削減します。 たとえば、すべてのエンド ホスト 30 の仮想マシンでは、IP の数の平均があり、ネットワーク インフラストラクチャに覚えさせる必要がある MAC アドレスが減少の倍数のパケットに埋め込まれた仮想サブネット Id も有効に実際の顧客にパケットを簡単に関連付ける。  
  
Windows Server 2012 R2 のスキームを共有 PA は、VSID あたり 1 つの PA をホストごとです。 Windows Server 2016 の NIC チームのメンバーごとに 1 つの PA は、スキームです。  
  
Windows Server 2016 以降 HNV 状態完全にサポート NVGRE と VXLAN ボックスです。アップグレードまたは Nic (ネットワーク アダプター)、スイッチ、ルーターなどの新しいネットワーク ハードウェアを購入することは必要ありません。 これは、ネットワーク上のこれらのパケットが現在のネットワーク インフラストラクチャと互換性のある PA 空間での通常の IP パケットであるためです。  ただし、パフォーマンスを最大限に利用を取得するには、タスク オフロードをサポートする最新のドライバーと Nic をサポートします。  
  
## <a name="multi-tenant-deployment-example"></a>マルチ テナント展開の例  
次の図は、CA と PA の関係をネットワーク ポリシーで定義されているクラウド データ センター内にある 2 つの顧客の展開例を示します。  
  
![マルチ テナント展開の例](../../../media/hyper-v-network-virtualization-technical-details-in-windows-server/VNetF5.png)  
  
図 8: マルチ テナント展開の例  
  
図 8 の例を検討してください。 前に、ホスティング プロバイダーへの移行の共有 IaaS サービス。  
  
-   Contoso Corp は、SQL Server を実行 (という名前の**SQL**)、ip アドレス 10.1.1.11 と web サーバー (という名前の**Web**)、SQL Server データベースのトランザクションを使用して、IP アドレス 10.1.1.12、します。  
  
-   Fabrikam Corp もという名前の SQL Server を実行した**SQL**し、IP アドレス 10.1.1.11 でともという名前の web サーバーを割り当てた**Web** 、IP アドレス 10.1.1.12 に使用して、SQL Server をデータベース トランザクションします。  
  
ホスティング サービス プロバイダーで、物理ネットワーク トポロジに対応するネットワーク コント ローラーを介してプロバイダー (PA) の論理ネットワークが作成済みと仮定されます。 ネットワーク コント ローラーは、ホストが接続されている論理サブネットの IP プレフィックスから 2 つの PA IP アドレスを割り当てます。 ネットワーク コント ローラーでは、IP アドレスを適用する適切な VLAN タグも表示されます。  
  
その仮想ネットワークとホスティング サービス プロバイダーによって指定されたプロバイダー (PA) の論理ネットワークでサブネットを作成し、ネットワーク コント ローラー、Contoso Corp と Fabrikam Corp を使用しています。 Contoso Corp と Fabrikam Corp は、それぞれの SQL Server を移動し、web サーバーを同じホスティング プロバイダーの共有 IaaS サービスでを偶然、 **SQL** HYPER-V Host 1 上のバーチャル マシン、および**Web** (IIS7) 仮想マシンを HYPER-V Host 2 です。 すべての仮想マシンは、元のイントラネット IP アドレス (各社の Ca) を維持します。  
  
両社は、以下に示すように、ネットワーク コント ローラーによって、次の仮想サブネット ID (VSID) を割り当てられます。  各 HYPER-V ホスト上のホストのエージェントは、ネットワーク コント ローラーから割り当てられている PA IP アドレスを受信し、既定ではないネットワーク コンパートメントの 2 つの PA ホスト Vnic を作成します。 ネットワーク インターフェイスは、各 PA IP アドレスが割り当てられている次のように、これらのホスト Vnic に割り当てられます。  
  
-   VSID と Pa の Contoso Corp の仮想マシン: **VSID** 5001、 **SQL の PA**は 192.168.1.10、 **Web の PA**は 192.168.2.20  
  
-   VSID と Pa の Fabrikam Corp の仮想マシン: **VSID**は 6001、 **SQL の PA**は 192.168.1.10、 **Web の PA**は 192.168.2.20  
  
ネットワーク コント ローラーでは、(OVSDB データベース テーブル) の固定ストアのポリシーを維持する SDN ホスト エージェントへ (CA と PA のマッピングを含む) すべてのネットワーク ポリシーが組み込まれます。  
  
HYPER-V ホスト 2 で Contoso Corp の Web 仮想マシン (10.1.1.12) は、10.1.1.11 で SQL Server への TCP 接続を作成するとき、次の処理が行われます。  
  
-   10.1.1.11 の宛先 MAC アドレスの VM Arp  
  
-   VFP 拡張を vSwitch では、このパケットをインターセプトし、SDN ホスト エージェントに送信します。  
  
-   SDN のホストのエージェントが 10.1.1.11 の MAC アドレスの場合は、そのポリシー ストアで検索します。  
  
-   MAC が見つかった場合、ホストのエージェントは、VM への ARP 応答を挿入します。  
  
-   MAC が見つからない場合は、応答は送信されず、10.1.1.11 用の VM で ARP エントリが到達不能でマークされています。  
  
-   正しい CA イーサネットと IP ヘッダーで TCP パケットを作成します。 し、vSwitch に送信できるようになりました、VM  
  
-   転送拡張機能、vSwitch VFP を通じて、VFP (下記参照) に割り当てられたレイヤー ソース vSwitch ポートをパケットを受信しましたし、VFP 統一されたフロー テーブルの新しいフローのエントリを作成には、このパケットを処理します。  
  
-   VFP エンジンは、IP およびイーサネット ヘッダーに基づく各レイヤー (仮想ネットワーク レイヤーなど) の規則に一致するまたはフロー テーブルの検索を実行します。  
  
-   仮想ネットワーク レイヤーに一致した規則は、CA と PA のマッピング領域を参照し、カプセル化を実行します。  
  
-   いずれか VXLAN (NVGRE) カプセル化の種類は、VSID と共に VNet レイヤーで指定されます。  
  
-   VXLAN のカプセル化の場合は、5001、VXLAN ヘッダー内の VSID と外側の UDP ヘッダーが構築されます。  
    HYPER-V Host 2 (192.168.2.20) と、SDN ホスト エージェントのポリシー ストアをそれぞれに基づいて HYPER-V ホスト 1 (192.168.1.10) に割り当てられている送信元と送信先の PA アドレスでは、外部の IP ヘッダーが構築されます。  
  
-   このパケットは、VFP の PA ルーティングのレイヤーに流れます。  
  
-   VFP で、PA ルーティング レイヤーは PA 空間のトラフィックと VLAN ID に使用するネットワーク コンパートメントを参照し、正しく PA パケットを HYPER-V Host 1 に転送するように、ホストの TCP/IP スタックを使用します。  
  
-   カプセル化されたパケットの受信、HYPER-V ホスト 1 PA ネットワーク コンパートメントのパケットを受信され、vSwitch に転送します。  
  
-   VFP はその VFP レイヤーを介してパケットを処理し、VFP 統一されたフロー テーブルに新しいフローのエントリを作成します。  
  
-   VFP エンジンは、仮想ネットワーク レイヤーで ingres 規則と一致して、特に外側のカプセル化されたパケットのイーサネット、IP、および VXLAN のヘッダーを取り除きます。  
  
-   VFP エンジンは、移行先の VM が接続されている vSwitch ポートにパケットを転送します。  
  
Fabrikam Corp 間のトラフィックに対して同様のプロセス**Web**と**SQL**仮想マシンは、Fabrikam 社の HNV ポリシー設定を使用その結果、Fabrikam Corp の HNV と Contoso Corp の仮想マシンの元のイントラネット上かのようを操作します。 ありません、同じ IP アドレスを使用している場合でも、相互に操作ができます。  
  
切り分けられたアドレス (Ca および Pa)、HYPER-V ホストのポリシー設定と、CA と仮想マシンの受信と送信トラフィックの PA 間のアドレス変換は、これらの NVGRE キーまたは VLXAN VNID のいずれかを使用してサーバーのセットを分離します。 さらに、仮想化のマッピングと変換は、仮想ネットワーク アーキテクチャが物理ネットワーク インフラストラクチャから切り離します。 Contoso **SQL**と**Web**と Fabrikam **SQL**と**Web**は独自の CA IP サブネット (10.1.1/24) 内に存在、物理的な展開がそれぞれ異なる PA サブネット、192.168.1/24 および 192.168.2/24、2 つのホストで行われます。 つまり、あるサブネット間で仮想マシンのプロビジョニングとライブ マイグレーションが可能になります HNV でです。  
  
## <a name="hyper-v-network-virtualization-architecture"></a>HYPER-V ネットワーク仮想化アーキテクチャ  
Windows Server 2016 HNVv2 は、Azure 仮想フィルタ リング プラットフォーム (VFP) は、HYPER-V スイッチ内の NDIS フィルター拡張を使用して実装されます。 VFP の重要な概念は、ネットワーク ポリシーのプログラミングの SDN ホスト エージェントに公開されている内部 API を使用してマッチ アクション フロー エンジンのことです。 SDN ホスト エージェント自体は、ネットワーク コント ローラーから OVSDB と WCF SouthBound の通信チャネル経由でネットワーク ポリシーを受け取ります。 ポリシーは仮想ネットワークをだけでなくです (例: CA と PA のマッピング) VFP が、Acl や QoS などの他のポリシーを使用してプログラムを作成します。  
  
オブジェクトの階層の vSwitch および VFP 転送拡張機能は次のよう  
  
-   vSwitch  
  
    -   外部 NIC の管理  
  
    -   NIC ハードウェア オフロード  
  
    -   グローバル転送ルール  
  
    -   ポート  
  
        -   出口髪ピン留めの層の転送  
  
        -   マッピングと NAT プールの容量が一覧表示します。  
  
        -   統一されたフロー テーブル  
  
        -   VFP レイヤー  
  
            -   フロー テーブル  
  
            -   グループ  
  
            -   ルール  
  
                -   ルールは、スペースを参照できます。  
  
レイヤーは、VFP でポリシーの種類 (たとえば、仮想ネットワーク) ごとに作成されますされ、汎用の一連のルール/フロー テーブル。 ないの組み込み機能まで、このような機能を実装する特定の規則がその層に割り当てられています。 各レイヤーには、優先順位が割り当てられているされ、レイヤーは、優先順位を昇順でポートに割り当てられます。 ルールは、方向と IP アドレス ファミリーに主に基づくグループに整理されます。 グループが優先順位が割り当てられているし、最大でグループから 1 つのルールは、特定のフローと一致します。  
  
VFP 拡張子を持つ vSwitch の転送のロジックは次のとおりです。  
  
-   入力処理 (ポートに着信パケットの観点から受信)  
  
-   転送  
  
-   出口処理 (ポートから出るパケットの観点から出口)  
  
VFP は、NVGRE と VXLAN カプセル化の種類だけでなく外部 MAC VLAN ベースの転送の MAC の内部の転送をサポートします。  
  
VFP 拡張機能は、低速パスとパケット トラバーサルの高速パスにあります。 最初のパケット フローでする必要がありますレイヤー各内のすべてのルール グループに走査し、規則の操作を行います負荷の高い操作を参照します。 ただし、(マッチ ルールに基づく) 操作の一覧と統合されたフロー テーブルのフローが登録されるとそれ以降のすべてのパケットが処理されます統一されたフロー テーブル エントリに基づいています。  
  
HNV のポリシーは、ホストのエージェントによって設定されます。 各仮想マシン ネットワーク アダプターは、IPv4 アドレスで構成されます。 これらの仮想マシンが相互に通信するために使用される Ca は、仮想マシンから IP パケットで伝送されます。 HNV は、ホストのエージェントのデータベースに格納されているネットワーク仮想化ポリシーに基づいて PA フレームで CA フレームをカプセル化します。  
  
![HNV のアーキテクチャ](../../../media/hyper-v-network-virtualization-technical-details-in-windows-server/VNetF7.png)  
  
図 9: HNV アーキテクチャ  
  
## <a name="summary"></a>概要  
クラウド ベースのデータ センターの強化されたスケーラビリティとリソース使用率の向上など、多くのメリットを提供できます。 これらの潜在的なメリットを実現するには、根本的に、動的環境におけるマルチ テナントのスケーラビリティの問題を解決するテクノロジが必要です。 HNV は、これらの問題を解決し、仮想ネットワーク トポロジが物理ネットワーク トポロジを分離することでも、データ センターの運用効率を向上させるように設計されました。 HNV は、既存の標準に基づいて構築、今日のデータ センターで実行され、既存の VXLAN インフラストラクチャと動作します。 HNV を持つユーザーでは、今すぐ自社のデータ センターをプライベート クラウドに統合したり、ハイブリッド クラウド ホスティング サーバー プロバイダーの環境に自社のデータ センターをシームレスに拡張することができます。  
  
## <a name="BKMK_LINKS"></a>参照してください。  
学習 HNVv2 について、次のリンクを参照してください。  
  
|コンテンツの種類|参照|  
|----------------|--------------|  
|**コミュニティ リソース**|-   [プライベート クラウド アーキテクチャのブログ](http://blogs.technet.com/b/privatecloud)<br />-質問します。[cloudnetfb@microsoft.com](mailto:%20cloudnetfb@microsoft.com)|  
|**RFC**|-   [NVGRE ドラフト RFC](http://www.ietf.org/id/draft-sridharan-virtualization-nvgre-07.txt)<br />-   [VXLAN の RFC 7348](http://www.rfc-editor.org/info/rfc7348)|  
|**関連するテクノロジ**|-Windows Server 2012 R2 で HYPER-V ネットワーク仮想化の技術的な詳細を参照してください[HYPER-V ネットワーク仮想化の技術的な詳細](https://technet.microsoft.com/library/jj134174.aspx)<br />-   [ネットワーク コント ローラー](../../../sdn/technologies/network-controller/Network-Controller.md)|  
  

