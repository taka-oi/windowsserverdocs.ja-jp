---
title: フェールオーバー クラスターでクラスターの共有ボリュームを使用します。
description: フェールオーバー クラスターでクラスターの共有ボリュームを使用する方法。
ms.prod: windows-server-threshold
ms.topic: article
author: JasonGerend
ms.author: jgerend
ms.technology: storage-failover-clustering
ms.date: 04/05/2018
ms.localizationpriority: medium
ms.openlocfilehash: f5bd0ad05bdc2573a5ea0abbe165de2d3e7f5c8f
ms.sourcegitcommit: 375e94dc70c76e7aa5679c32f0f4dbc26cf7dd83
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/10/2018
ms.locfileid: "2233518"
---
# <a name="use-cluster-shared-volumes-in-a-failover-cluster"></a>フェールオーバー クラスターでクラスターの共有ボリュームを使用します。

>対象: Windows Server 2012 R2、Windows Server 2012、Windows Server 2016

クラスター共有ボリューム (CSV) では、複数のノードを同時に同じ lun (ディスク) として、ボリュームが自動的にプロビジョニングの読み取り/書き込みのアクセス権を持つフェールオーバー クラスターで有効にします。 (Windows Server 2012 R2 のディスク プロビジョニングできる NTFS と回復ファイル システム (参照)。)Csv 形式] で集合役割が失敗をすばやく 1 つのノードからを別のノード マウントやボリュームを再マウント ドライブの所有者権限を変更することがなくします。 CSV では、フェールオーバー クラスターで Lun の数が多い可能性があるの管理を簡素化できます。

CSV、NTFS (または Windows Server 2012 R2 の参照) の上に重なっている汎用、集合ファイル システムを提供します。 CSV のアプリケーションは、次のとおりです。

- 集合集合 HYPER-V 仮想マシンの仮想ハード_ディスク (VHD) ファイル
- スケール アウト ファイル サーバーの集合役割のアプリケーションのデータを格納するスケール アウト ファイル共有します。 この役割のアプリケーションのデータの例では、HYPER-V 仮想マシンのファイルと Microsoft SQL Server データを表示します。 (にスケール アウト ファイル サーバーの参照がサポートされていないことに注意してください)。スケール アウト ファイル サーバーの詳細については、[アプリケーションのデータのスケール アウト ファイル サーバー](sofs-overview.md)を参照してください。

>[!NOTE]
>CSV は、SQL Server 2012、SQL Server の以前のバージョンの Microsoft SQL Server の集合作業負荷をサポートしていません。

Windows Server 2012、CSV 機能が大幅に向上します。 たとえば、Active Directory ドメイン サービスの依存関係が削除されました。 **Chkdsk**の機能の改善点、ウイルス対策、およびバックアップのアプリケーションと相互運用性と BitLocker 暗号化ボリュームやストレージ スペースなどの一般的な記憶域の機能との統合にサポートが追加されました。 Windows Server 2012 で導入された CSV 機能の概要については、次を参照してください。[では、Windows Server 2012 \[redirected\ しないでの新機能]](<https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/dn265972(v%3dws.11)>)します。

Windows Server 2012 R2 分散 CSV 所有権、サーバーのサービスをより柔軟により、CSV キャッシュに割り当てることができる物理メモリ容量の空き時間情報を復元機能が強化などの他の機能を紹介します。diagnosibility と相互運用性の拡張を参照し、重複のサポートを含みます。 詳細については[しないでの新機能](<https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/dn265972(v%3dws.11)>)」を参照してください。

>[!NOTE]
>CSV の仮想デスクトップ インフラストラクチャ (VDI) シナリオのデータの重複を使用する方法の詳細についてを参照してください、ブログ投稿の[Windows Server 2012 R2 の VDI 記憶域を展開するデータが重複除外](https://blogs.technet.com/b/filecab/archive/2013/07/31/deploying-data-deduplication-for-vdi-storage-in-windows-server-2012-r2.aspx)と[拡張するデータの重複で新しいワークロードWindows Server 2012 R2](https://blogs.technet.com/b/filecab/archive/2013/07/31/extending-data-deduplication-to-new-workloads-in-windows-server-2012-r2.aspx)します。

## <a name="review-requirements-and-considerations-for-using-csv-in-a-failover-cluster"></a>要件およびフェールオーバー クラスターで CSV を使用するための考慮事項を確認します。

フェールオーバー クラスターで CSV を使用する前に、ネットワーク、ストレージ、およびその他の要件およびこのセクションの考慮事項を確認します。

### <a name="network-configuration-considerations"></a>ネットワークの構成に関する考慮事項

CSV をサポートするネットワークを構成する場合は、次を検討してください。

- **複数のネットワークと複数のネットワーク アダプター**を表示します。 ネットワークのエラーが発生した場合のフォールト トレランスを有効にするには、複数のクラスター ネットワーク CSV トラフィックのネットワーク アダプターをチームを構成することにお勧めします。
    
    クラスター ノードは、クラスターで使用する必要がないネットワークに接続された、それらが無効にする必要があります。 たとえば、これらのネットワーク上の CSV トラフィックを防ぐためにクラスター用 iSCSI ネットワークを無効にすることをお勧めします。 フェールオーバー クラスター マネージャーで、ネットワークを無効にするには、**ネットワーク**を選択、ネットワークを選択して、**プロパティ**をクリックし、[**このネットワーク上のクラスター ネットワークの通信を許可しない**] を選びます。 または、 [Get ClusterNetwork](https://docs.microsoft.com/powershell/module/failoverclusters/get-clusternetwork?view=win10-ps)の Windows PowerShell コマンドレットを使用して、ネットワークの**役割**のプロパティを設定できます。
- **ネットワーク アダプターのプロパティ**。 クラスター通信に対応するすべてのアダプターのプロパティ] で、次の設定が有効になっていることを確認します。

  - **Microsoft ネットワーク クライアント**と**ファイルとプリンターの Microsoft ネットワーク共有**します。 これらの設定は、サーバー メッセージ ブロック (一般法) 3.0、ノード間で CSV トラフィックを実行するために、既定で使用されるをサポートします。 一般法を有効にするには、サーバーのサービスとワークステーション サービスが実行されていると、クラスター ノードごとに自動的に開始する構成されていることをも確認します。

    >[!NOTE]
    >Windows Server 2012 R2 クラスター ノードあたり Server サービスの複数のインスタンスがあります。 通常のファイル共有へのアクセス一般法クライアントから受信トラフィックを処理する既定のインスタンス、2 番目の CSV インスタンスのみのノード間 CSV トラフィックを処理します。 また、ノードのサーバーのサービスが問題になると、CSV 所有権は自動的に別のノードに移行します。

    一般法 3.0 には、リモート ダイレクト メモリ アクセス (RDMA) をサポートするネットワーク アダプターを活用してクラスターで複数のネットワークをストリーム CSV トラフィックを有効にする一般法マルチ チャンネルと直接一般法機能が含まれています。 既定では、CSV トラフィック用一般法マルチ チャンネルを使用します。 詳細については、[サーバー メッセージ ブロックの概要](../storage/file-server/file-server-smb-overview.md)を参照してください。
  - **Microsoft フェールオーバー仮想アダプター パフォーマンス フィルター**します。 この設定は、CSV ディスクに直接接続ノードの接続に失敗には、たとえば、CSV に達するまでに必要な場合は、I/O のリダイレクトを実行するノードの機能が向上します。 詳細については、このトピックの後半[の I/O 同期および CSV 通信 I/O リダイレクト](#about-i/o-synchronization-and-i/o-redirection-in-csv-communication)を参照してください。
- **クラスター ネットワークの優先順位付け**します。 一般に、ネットワークのクラスター構成の環境設定を変更しないことをお勧めします。
- **IP サブネット構成**します。 特定のサブネットの構成の CSV を使用しているネットワークでのノードの必要はありません。 CSV は、マルチ サブネットのクラスターをサポートできます。
- **ポリシー ベース サービスの品質 (QoS)**。 CSV を使用すると、QoS 優先度のポリシーと各ノードのネットワーク トラフィックの帯域幅のポリシーを構成することをお勧めします。 詳細については、[サービスの品質 (QoS)](<https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh831679(v%3dws.11)>)を参照してください。
- **記憶域ネットワーク**します。 記憶域ネットワークの推奨事項については、記憶域の製造元によって提供されるガイドラインを確認します。 CSV の記憶域に関する考慮事項を追加、このトピックの後半[の記憶領域やディスクの構成要件](#storage-and-disk-configuration-requirements)を参照してください。

ハードウェア、ネットワーク、およびフェールオーバー クラスターの記憶域の要件の概要については、[フェールオーバー クラスターのハードウェア要件と記憶域のオプション](clustering-requirements.md)を参照してください。

#### <a name="about-io-synchronization-and-io-redirection-in-csv-communication"></a>I/O 同期と通信で CSV I/O のリダイレクトについて

- **I/O 同期**: CSV、同じ共有記憶域に読み取り/書き込みアクセスを同時に複数のノードを有効にします。 ノードは、CSV ボリューム ディスク入出力 () を実行するときにノード直接と通信、ストレージ、たとえば、記憶域エリア ネットワーク (SAN) を使ってします。 ただし、いつでも、1 つのノード (調整役のノードと呼ばれる)「リソースを所有して」物理ディスク LUN に関連付けられています。 CSV の音量を調整役のノードには、[**ディスク**の**所有者ノード**としてフェールオーバー クラスター マネージャーでが表示されます。 [Get ClusterSharedVolume](https://docs.microsoft.com/powershell/module/failoverclusters/get-clustersharedvolume?view=win10-ps)の Windows PowerShell コマンドレットの出力にも表示されます。

  >[!NOTE]
  >Windows Server 2012 R2 では、各ノードを所有している CSV ボリューム数に基づいてフェールオーバー クラスター ノードの間で CSV 所有権が均等に分散します。 さらに、所有権は CSV フェールオーバーなどの条件がある、ノード クラスターに再び参加、クラスターに新しいノードを追加する、クラスター ノードを再起動する、またはシャット ダウンした後、フェールオーバーを開始するときに自動的に不整合です。

  特定の小さな変更は、CSV ボリューム上のファイル システムで発生した場合、このメタデータがの 1 つの調整役のノードのだけでなく、LUN にアクセスする物理ノードごとに同期されている必要があります。 たとえば、CSV ボリューム上の仮想マシンの開始、作成または削除すると、または仮想マシンの移行がある場合、この情報を各物理ノード仮想マシンにアクセスするのに同期する必要があります。 これらのメタデータ更新操作では、一般法 3.0 を使用してクラスター ネットワーク経由で並列で発生します。 これらの操作には、共有ストレージと通信するすべての物理ノードは不要です。

- **I/O リダイレクト**: 記憶域接続が失敗し、特定の記憶域操作できなくなる特定のノード ストレージと直接通信します。 関数を維持しますが、記憶域が通信していないときに、ノードはクラスター ネットワーク経由で I/O、ディスクをディスクが現在マウント調整役のノードにリダイレクトします。 現在調整役のノードに記憶域接続エラーが発生した場合、新しいノードは調整役のノードとして設定されたときに、一時的にすべてのディスク I/O 操作がキューにあります。

サーバーでは、状況によっては、次の I/O リダイレクション モードのいずれかを使用します。

- **ファイル システムのリダイレクト**リダイレクションが 1 つのボリュームが: たとえばと CSV スナップショットはバックアップ アプリケーションでリダイレクト I/O モードに CSV ボリュームを手動で配置します。
- **ブロックのリダイレクト**ファイル制限機能のレベルでは、リダイレクション-ストレージへの接続がボリュームが失われた場合などです。 ブロックのリダイレクトがファイル システムのリダイレクトよりも大幅に短縮します。

Windows Server 2012 R2 のノードごとに CSV ボリュームの状態を表示できます。 たとえば、またはかどうか I/O 直接的またはリダイレクトされた場合は、CSV ボリュームが利用できるかどうかを表示できます。 CSV のボリュームが I/O リダイレクト モードの場合は、理由を表示することもできます。 この情報を表示するのにには、Windows PowerShell コマンドレット**Get ClusterSharedVolumeState**を使用します。

>[!NOTE]
> * Windows Server 2012、CSV の [デザイン] での改善点の理由により、CSV 操作を実行直接 I/O モードで Windows Server 2008 R2 で発生します。
> * CSV の一般法 3.0 などの機能一般法マルチ チャンネル一般法直接との統合の理由により、リダイレクト I/O トラフィックを複数のクラスター ネットワーク ストリームできます。
> * 調整役のノードへのネットワーク トラフィックの増加に潜在的な I/O リダイレクト中にできるクラスター ネットワークを計画する必要があります。

### <a name="storage-and-disk-configuration-requirements"></a>ストレージとディスクの構成の要件

CSV を使用するのには、ディスク、記憶域は、次の要件を満たす必要があります。

- **ファイル システムの形式**です。 Windows Server 2012 R2、ディスクやストレージ スペース CSV ボリュームの基本的なディスク NTFS または参照に分割されている必要があります。 Windows Server 2012、CSV ボリュームのディスクやストレージ スペース基本的なディスク NTFS でパーティションされている必要があります。

  CSV には、次の他の要件があります。
    
  - Windows Server 2012 R2 で FAT または FAT32 で書式設定されている CSV のディスクを使用することはできません。
  - Windows Server 2012、FAT、FAT32、または参照を使用して書式設定は、CSV のディスクを使用することはできません。
  - CSV、記憶域を使用する場合は、または見開き単純なスペースを構成できます。 Windows Server 2012 R2 の同じ場所を構成することもできます。 (Windows Server 2012、CSV はサポートしません同じスペース。)
  - CSV はクォーラム補助ディスクとしては使用できません。 クラスター クォーラムの詳細については、[記憶域スペース直接についてクォーラム](../storage/storage-spaces/understand-quorum.md)を参照してください。
  - Csv 形式ディスクを追加した後はその CSV ファイル システム) (CSVFS 形式で指定されます。 これにより、クラスターとその他のソフトウェアを他の NTFS から CSV 記憶域または記憶域の参照を区別するためにします。 通常、CSVFS NTFS または参照と同じ機能がサポートしています。 ただし、特定の機能はサポートされません。 たとえば、Windows Server 2012 R2 で CSV での圧縮を有効にすることはできません。 Windows Server 2012 では、データの重複または CSV での圧縮を有効にすることはできません。
- **クラスターでリソースの種類**です。 CSV ボリュームの物理ディスク リソースの種類を使用する必要があります。 既定では、次のように、クラスター記憶域に追加されるディスクやストレージ スペースを自動的に構成します。
- **選択肢の CSV ディスクまたはその他のディスク クラスター記憶域に**します。 集合仮想マシンの 1 つまたは複数のディスクを選択する場合は、各ディスクの使用方法を検討してください。 構成ファイル、VHD ファイルなど、HYPER-V によって作成されたファイルを保存するディスクを使用する場合は、CSV ディスクまたはクラスター記憶域に使用できる他のディスクから選択できます。 直接は実際のディスクをディスクにする場合、仮想マシン (パススルー ディスクとも呼ばれます) に接続されている CSV ディスク] を選択することはできず、クラスター記憶域の利用可能なその他のディスクから選ぶ必要があります。
- **ディスクを識別するためのパス名**です。 CSV のディスクは、パス名で識別されます。 各パスは、 **\\ClusterStorage**フォルダーの下の段落番号付きのボリュームとしてノードのシステム ドライブ上に表示されます。 このパスは、クラスター内の任意のノードから表示されると同じです。 必要な場合は、ボリュームの名前を変更することができます。

CSV の記憶域要件については、記憶域の製造元によって提供されるガイドラインを確認します。 追加の記憶域 CSV の計画に関する考慮事項、このトピックの後半で説明[するフェールオーバーで CSV の使用を計画する](#plan-to-use-csv-in-a-failover-cluster)を参照してください。

### <a name="node-requirements"></a>ノードの要件

CSV を使用するには、ノードは、次の要件を満たす必要があります。

- **システムのディスク ドライブ**をします。 すべてのノードのシステムのディスク ドライブ必要がありますと同じです。
- **認証プロトコル**です。 すべてのノードの NTLM プロトコルを有効にする必要があります。 これは既定で有効です。

## <a name="plan-to-use-csv-in-a-failover-cluster"></a>フェールオーバー クラスターで CSV の使用を計画するには

ここでは、計画の考慮事項と Windows Server 2012 R2 または Windows Server 2012 を実行してフェールオーバーで CSV を使用するための推奨事項を示します。

>[!IMPORTANT]
>CSV の特定の記憶域の単位を構成する方法については、推奨事項については、記憶域の製造元を問い合わせてください。 記憶域仕入先からの推奨事項は、このトピックの情報と異なる場合、記憶域仕入先からの推奨事項を使用します。

### <a name="arrangement-of-luns-volumes-and-vhd-files"></a>Lun、ボリューム、および VHD ファイルの配置

集合仮想マシンの記憶域を提供する CSV を最大限に活用するために、物理サーバーを構成するときに Lun (ディスク) の配置と方法を確認することをお勧めします。 対応する仮想マシンを構成する場合は、同様の方法で VHD ファイルを整理してみてください。

物理サーバーを配置することがディスクとファイル次のように検討してください。

- 物理ディスク、ページのファイルなどのシステム ファイル
- 別の物理ディスク上にデータ ファイル

サポート会社と同等の集合仮想マシンと同じようにボリュームおよびファイルを整理する必要があります。

- ページ、ファイルを 1 つの CSV ファイルを VHD でなどのシステム ファイル
- 別の CSV ファイルを VHD 内のデータ ファイル

可能であれば、別の仮想マシンを追加する場合は、その仮想マシンの同じ Vhd の配置を維持すべきします。

### <a name="number-and-size-of-luns-and-volumes"></a>Lun とボリュームの数とサイズ

CSV を使用しているフェールオーバー クラスターの記憶域の構成を計画する場合は、以下の推奨事項を考慮します。

- 構成するのには Lun の数を決定するには、記憶域のベンダーを参照してください。 たとえば、記憶域のベンダーには 1 つのパーティションを持つ各 LUN を構成し、1 つの CSV ボリュームを配置することがお勧めします。
- 1 つの CSV ボリュームでサポートされる仮想マシンの数に制限はありません。 ただし、クラスターで各仮想マシンの作業負荷 (I/O 操作/秒) を計画する仮想マシンの数を検討してください。 次に例を示します。

  - 1 つの組織がサポートは比較的負荷仮想デスクトップ インフラストラクチャ (VDI)、仮想マシンを展開します。 クラスター ハイ パフォーマンスの記憶域を使用します。 クラスター管理者では、記憶域のベンダーに相談後に、比較的多数の CSV ボリュームあたりの仮想マシンの配置を決定します。
  - 別の組織が重い作業負荷を使用頻度の高いデータベース アプリケーションをサポートする仮想マシンの数が多いを展開します。 クラスターでは、下の操作を実行する記憶域を使用します。 クラスター管理者では、記憶域のベンダーに相談後に、比較的少数の CSV ボリュームあたりの仮想マシンの配置を決定します。
- 特定の仮想マシンの記憶域の構成を計画する場合は、サービス、アプリケーション、または仮想マシンでサポートされるロールのディスク要件を考慮します。 これらの要件を理解すると、パフォーマンスが低下につながるディスクの競合を避けることができます。 仮想マシンの記憶域の構成の記憶域の構成で同じようなサービス、アプリケーション、または役割が実行されている物理サーバーを使用するようになります密接にします。 詳細については、このトピックの「 [Lun の配置、ボリューム、および VHD ファイル](#arrangement-of-luns,-volumes,-and-vhd-files)を参照してください。

    ストレージを独立した物理ハード ディスクの数が多いとするもディスクの競合を軽減できます。 したがって、記憶域ハードウェアを選択し、記憶域のパフォーマンスを最適化するには、製造元に問い合わせてくださいします。
- クラスター ワークロード、I/O 操作の必要に応じて他の仮想マシンに接続するいないと、専用の操作を計算するためには、代わりに、各 LUN にアクセスする仮想マシンのパーセントだけを構成するを検討することができます。

## <a name="add-a-disk-to-csv-on-a-failover-cluster"></a>Csv フェールオーバー クラスターでディスクを追加します。

CSV 機能はしないでに既定で有効になります。 CSV には、ディスクを追加、ディスクをクラスター (は既に追加されていない場合) の**利用可能なストレージ**グループに追加し、csv クラスターのディスクを追加して必要があります。 これらの手順を実行するのには、フェールオーバー クラスター マネージャーまたはフェールオーバー クラスターの Windows PowerShell コマンドレットを使用できます。

### <a name="add-a-disk-to-available-storage"></a>ディスクを利用可能なストレージに追加します。

1. フェールオーバー クラスター マネージャーを使用して、コンソール ツリーでは、クラスターの名前を展開し、[**記憶域**] を展開します。
2. **ディスク**を右クリックし、[**ディスクの追加**] を選びます。 フェールオーバー クラスターで使用するために追加できるディスクが表示されているリストが表示されます。
3. または、追加する複数のディスクを選択し、[ **ok]** をクリックします。

    ディスクが**利用可能なストレージ**グループに割り当てられているようになりました。

#### <a name="windows-powershell-equivalent-commands-add-a-disk-to-available-storage"></a>Windows PowerShell と同じコマンド (使用可能な記憶域にディスクを追加する)

次の Windows PowerShell コマンドレットまたはコマンドレットは、上記の手順と同じ機能を実行します。 表示される場合でもが複数の行には、ここにわたるワード ラップ制約を書式設定の理由により、1 行の場合は、各コマンドレットを入力します。

次の例では、クラスターに追加する準備ができているディスクを識別し、[**利用可能なストレージ**グループに追加します。

```PowerShell
Get-ClusterAvailableDisk | Add-ClusterDisk
```

### <a name="add-a-disk-in-available-storage-to-csv"></a>利用可能な記憶域に csv ディスクを追加します。

1. フェールオーバー クラスター マネージャーを使用して、コンソール ツリーでは、クラスターの名前を展開、**記憶域**を展開し、[**ディスク**を選択します。
2. いずれかを選ぶか、ディスクを**利用可能な記憶域**の割り当てられているが、選択範囲を右クリックし、**クラスターの共有ボリュームに追加**] をクリックします。

    ディスクがクラスターで**クラスター共有ボリューム**グループに割り当てられているようになりました。 ディスクは、SystemDisk ClusterStorage フォルダーの下の段落番号付きのボリューム (マウント ポイント) として、各クラスター ノードに公開されます。 CSVFS ファイル システム内のボリュームが表示されます。

>[!NOTE]
>CSV ボリューム SystemDisk ClusterStorage フォルダーの名前を変更することができます。

#### <a name="windows-powershell-equivalent-commands-add-a-disk-to-csv"></a>Windows PowerShell と同じコマンド (csv、ディスクを追加する)

次の Windows PowerShell コマンドレットまたはコマンドレットは、上記の手順と同じ機能を実行します。 表示される場合でもが複数の行には、ここにわたるワード ラップ制約を書式設定の理由により、1 行の場合は、各コマンドレットを入力します。

次の例は、**使用可能な記憶域**に*クラスター ディスク 1*をローカルのクラスターで CSV に追加します。

```PowerShell
Add-ClusterSharedVolume –Name "Cluster Disk 1"
```

## <a name="enable-the-csv-cache-for-read-intensive-workloads-optional"></a>(省略可能) の読み取り中心ワークロードの CSV キャッシュを有効にします。

CSV キャッシュの書き込みをキャッシュとしてシステム メモリ (RAM) を割り当てることによってブロック レベル読み取り専用のバッファー入出力のキャッシュを提供します。 (バッファー I/O 操作はキャッシュ マネージャーがキャッシュされません)。これにより、仮想ハード_ディスクにアクセスするときにバッファー I/O 操作を行っている HYPER-V などのアプリケーションのパフォーマンスが向上します。 CSV キャッシュは、書き込み要求のキャッシュを使用しない読み取り要求のパフォーマンスを改善することができます。 CSV のキャッシュを有効にするともスケール アウト ファイル サーバーの場合に便利です。

>[!NOTE]
>すべてのクラスター HYPER-V とスケール アウト ファイル サーバーの展開について CSV キャッシュを有効にすることをお勧めします。

Windows Server 2012 で、既定では、CSV キャッシュが無効です。 Windows Server 2012 R2 CSV キャッシュは既定で有効です。 ただし、予約するブロック キャッシュのサイズをまだ割り当てる必要があります。

次の表では、CSV キャッシュを制御する 2 つの構成の設定について説明します。

<table>
<thead>
<tr class="header">
<th>Windows Server 2012 R2 のプロパティ名</th>
<th>Windows Server 2012 でプロパティ名</th>
<th>説明</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>BlockCacheSize</strong></td>
<td><strong>SharedVolumeBlockCacheSizeInMB</strong></td>
<td>これは、クラスターの一般的なプロパティをクラスター内の各ノードの CSV キャッシュ用に予約するメガバイト単位でメモリの量を定義することができます。 たとえば、512 の値を定義すると場合、[512 MB のシステム メモリは各ノードの予約されています。 (多くのクラスターでは 512 MB を推奨される値は。)既定の設定は 0 (無効) について</td>
</tr>
<tr class="even">
<td><strong>EnableBlockCache</strong></td>
<td><strong>CsvEnableBlockCache</strong></td>
<td>これは、プライベート クラスター物理ディスク リソースのプロパティ。 CSV に追加される個々 の上に CSV キャッシュを有効にすることができます。 Windows Server 2012、既定の設定は 0 (無効) について ディスクの CSV キャッシュを有効にするには、1 の値を構成します。 Windows Server 2012 R2] で、既定では、この設定が有効にします。</td>
</tr>
</tbody>
</table>

**クラスター CSV ボリューム キャッシュ**のカウンターを追加して、パフォーマンス モニターの CSV キャッシュを監視できます。

#### <a name="configure-the-csv-cache"></a>CSV のキャッシュを構成します。

1. 管理者として、Windows PowerShell を起動します。
2. *512* MB を各ノードの予約のキャッシュを定義するには、次のように入力します。

    - Windows Server 2012 R2: 用

        ```PowerShell
        (Get-Cluster).BlockCacheSize = 512  
        ```

    - Windows server 2012 の場合。

        ```PowerShell
        (Get-Cluster).SharedVolumeBlockCacheSizeInMB = 512  
        ```
3. Windows Server 2012、*クラスター ディスク 1*という名前の CSV CSV キャッシュを有効にする入力します。

    ```PowerShell
    Get-ClusterSharedVolume "Cluster Disk 1" | Set-ClusterParameter CsvEnableBlockCache 1
    ```

>[!NOTE]
> * Windows Server 2012、のみ 20% の合計の RAM CSV キャッシュに割り当てることができます。 Windows Server 2012 R2 を 80% を割り当てることができます。 スケール アウト ファイル サーバーではない通常メモリ制約があるために、CSV キャッシュの余分なメモリを使用して大規模なパフォーマンスの向上を行うことができます。
> * リソースの競合を避けるためには、CSV キャッシュに割り当てられているメモリを変更した後クラスターで各ノードを再起動する必要があります。 Windows Server 2012 R2 で再起動する必要がなくなったします。
> * 有効にする、または CSV キャッシュ設定を有効になるまでの個々 のディスクを無効にした後は、物理ディスク リソースをオフラインし、オンラインに戻す必要があります。 (既定では、Windows Server 2012 R2 の CSV キャッシュは有効です。) 
> * パフォーマンス カウンターに関する情報を含む CSV キャッシュの詳細については、ブログの投稿[CSV キャッシュを有効にする方法](https://blogs.msdn.microsoft.com/clustering/2013/07/19/how-to-enable-csv-cache/)を参照してください。

## <a name="back-up-csv"></a>CSV をバックアップします。

フェールオーバー クラスターで CSV に保存されている情報をバックアップする複数の方法があります。 Microsoft バックアップ アプリケーションまたはサードパーティ製アプリケーションを使用することができます。 一般に、CSV は NTFS または参照で書式設定された集合の記憶域のだけでなく、特別なバックアップ要件を適用できません。 CSV のバックアップもが中断されないその他の CSV 記憶域操作します。

バックアップ アプリケーションと CSV のバックアップ スケジュールを選択するには、次の点を考慮してください。

- CSV ボリュームに接続する任意のノードから CSV ボリュームのボリューム レベルのバックアップを実行できます。
- ハードウェア スナップショット ソフトウェアや、バックアップ、アプリケーションを使用できます。 サポートしてバックアップ アプリケーションの機能、によっては、バックアップはアプリケーション一貫性のあると、クラッシュ一貫性のあるボリューム シャドウ コピー サービス (VSS) のスナップショットを使用できます。
- 複数の実行中の仮想マシン CSV をバックアップする場合は、一般的に、管理オペレーティング システム ベースのバックアップ方法を選択してください。 バックアップ アプリケーションでサポートされている場合は、複数の仮想マシンをバックアップできます同時にします。
- CSV では、Windows Server 2012 R2 バックアップ、Windows Server 2012 バックアップまたは Windows Server 2008 R2 バックアップを実行しているバックアップ リクエスターをサポートします。 ただし、Windows Server バックアップ通常のみ基本的なバックアップ ソリューションを提供クラスターが組織に適していない可能性があります。 Windows Server バックアップはサポートしていませんアプリケーション一貫性のある仮想マシンのバックアップ CSV にします。 クラッシュ一貫性のあるボリューム レベル バックアップのみがサポートしています。 (クラッシュ一貫性のあるバックアップを復元すると、仮想マシンなりますだったバックアップを実行する正確な時点で仮想マシンがクラッシュしたかどうかと同じ状態にします。)CSV ボリューム上の仮想マシンのバックアップを成功しますが、サポートされていないことを示すエラー イベントが記録されます。
- フェールオーバー クラスターをバックアップするときに、管理者の資格情報を必要があります。

>[!IMPORTANT]
>どのようなデータを慎重に確認するバックアップ アプリケーションをバックアップ、CSV 機能を復元し、ごとに、アプリケーションのリソース要件クラスター ノード サブドメインします。

>[!WARNING]
>CSV ボリュームにバックアップ データを復元する必要がある場合は、機能とを維持およびクラスター ノード間でアプリケーション一貫性のあるデータを復元するバックアップ アプリケーションの制限事項を把握します。 たとえば、一部のアプリケーションで CSV 復元される場合は、CSV 音量のバックアップの設定] ノードとは異なるノードの可能性があります誤ってを上書きする復元が行われているノードのアプリケーションの状態に関する重要なデータします。

## <a name="more-information"></a>詳細

- [フェールオーバー クラスタリング](failover-clustering.md)
- [集合記憶域を展開します。](<https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/jj822937(v%3dws.11)>)