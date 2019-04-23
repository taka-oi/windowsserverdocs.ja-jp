---
title: 記憶域スペース ダイレクトでのフォールト トレランスと記憶域の効率
ms.prod: windows-server-threshold
ms.author: cosmosdarwin
ms.manager: eldenc
ms.technology: storage-spaces
ms.topic: article
author: cosmosdarwin
ms.date: 10/11/2017
ms.assetid: 5e1d7ecc-e22e-467f-8142-bad6d82fc5d0
description: ミラーリングとパリティを含む記憶域スペース ダイレクトにおける回復性オプションの説明。
ms.localizationpriority: medium
ms.openlocfilehash: 4e6a29e82a85ec9570cda827060dfe1cdf192c53
ms.sourcegitcommit: 1533d994a6ddea54ac189ceb316b7d3c074307db
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/01/2018
ms.locfileid: "1284980"
---
# <a name="fault-tolerance-and-storage-efficiency-in-storage-spaces-direct"></a>記憶域スペース ダイレクトでのフォールト トレランスと記憶域の効率

>適用先: Windows Server 2016

このトピックでは、[記憶域スペース ダイレクト](storage-spaces-direct-overview.md)で利用できる回復オプションを紹介し、規模に関する要件、記憶域の効率、一般的な利点、およびそれぞれのトレードオフの概要について説明します。 また、作業を開始するための使用方法についても説明します。このトピックでは、詳細を確認する際に役立つ優れたドキュメントやブログなどのコンテンツを参照しています。

記憶域スペースの詳細を既に理解している場合は、「[まとめ](#summary)」セクションに進んでもかまいません。

## <a name="overview"></a>概要

本来、記憶域スペースの目的は、データのフォールト トレランス ("回復性" と呼ばれる場合もあります) を実現することです。 実装方法は RAID に似ていますが、複数のサーバーに分散され、ソフトウェアで実装される点が異なります。

RAID と同様に、記憶域スペースの実装方法には複数の方法があり、それぞれの方法では、フォールト トレランス、記憶域の効率、および計算の複雑さの間で発生するトレードオフが異なります。 これらの方法は、大まかに 2 つのカテゴリ "ミラーリング" と "パリティ" に分類されます。"パリティ" は、"イレイジャー コーディング (消失訂正符号)" と呼ばれる場合もあります。

## <a name="mirroring"></a>ミラーリング

ミラーリングでは、すべてのデータのコピーを複数維持することでフォールト トレランスが実現されます。 これは、RAID-1 に非常に類似しています。 データのストライプ方法や配置方法は重要となりますが (詳しくは[このブログ](https://blogs.technet.microsoft.com/filecab/2016/11/21/deep-dive-pool-in-spaces-direct/)をご覧ください)、ミラーリングを使用して保存されたすべてのデータが、そのまま複数回書き込まれるということも重要な点です。 各コピーは個別の物理ハードウェア (異なるサーバーの異なるドライブ) に書き込まれます。これは、各ハードウェアでエラーが発生した場合を想定した措置です。

Windows Server 2016 では、記憶域スペース用に 2 種類のミラーリング ("双方向" と "3 方向") が用意されています。

### <a name="two-way-mirror"></a>双方向ミラー

双方向ミラーリングでは、すべてのデータについて 2 つのコピーが書き込まれます。 その記憶域の効率は 50% です。つまり、1 TB のデータを書き込むには、少なくとも 2 TB の物理的な記憶域の容量が必要になります。 同様に、少なくとも 2 つの[ハードウェア "障害ドメイン"](../../failover-clustering/fault-domains.md) が必要になります (記憶域スペース ダイレクトでは 2 台のサーバーを意味します)。

![双方向ミラー](media/Storage-Spaces-Fault-Tolerance/two-way-mirror-180px.png)

   >[!WARNING]
   > サーバーが 3 台以上の場合は、代わりに 3 方向ミラーリングを使うことをお勧めします。

### <a name="three-way-mirror"></a>3 方向ミラー

3 方向ミラーリングでは、すべてのデータについて 3 つのコピーが書き込まれます。 その記憶域の効率は 33.3% です。つまり、1 TB のデータを書き込むには、少なくとも 3 TB の物理的な記憶域の容量が必要になります。 同様に、少なくとも 3 つのハードウェア障害ドメインが必要になります (記憶域スペース ダイレクトでは 3 台のサーバーを意味します)。

3 方向ミラーリングでは、[一度に少なくとも 2 件のハードウェア問題 (ドライブやサーバー)](#examples) に安全に対処できます。 たとえば、あるサーバーを再起動しているときに別のドライブやサーバーで突然障害が発生した場合、すべてのデータの安全性が保たれ、アクセスし続けることができます。

![3 方向ミラー](media/Storage-Spaces-Fault-Tolerance/three-way-mirror-180px.png)

## <a name="parity"></a>パリティ

パリティ エンコード ("イレイジャー コーディング (消失訂正符号)" と呼ばれる場合もあります) では、ビット単位の演算を使用してフォールト トレランスが実現されるため、[複雑さが大幅に増す可能性があります](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/LRC12-cheng20webpage.pdf)。 パリティのしくみはミラーリングよりわかりにくいですが、パリティを理解する際に役立つ、優れたオンライン リソースが多数あります (たとえば、サード パーティの「[Dummies Guide to Erasure Coding](http://smahesh.com/blog/2012/07/01/dummies-guide-to-erasure-coding/)」(イレイジャー コーディングのダミーガイド) など)。 あえて言うなら、パリティではフォールト トレランスを損なうことなく、優れた記憶域の効率が実現されます。

Windows Server 2016 では、記憶域スペース用に 2 種類のパリティ ("シングル" パリティと "デュアル" パリティ) が用意されています。"デュアル" パリティでは、大規模な場合に対応するための "ローカル再構築コード" と呼ばれる高度な手法が採用されています。

> [!IMPORTANT]
> ほとんどのパフォーマンスが重視されるワークロードには、ミラーリングを使用することをお勧めします。 ワークロードに応じて、パフォーマンスと容量のバランスを取る方法については詳しくは、[ボリュームの計画](plan-volumes.md#choosing-the-resiliency-type)」をご覧ください。

### <a name="single-parity"></a>シングル パリティ
シングル パリティでは、ビット単位のパリティ シンボルが 1 つだけ保持されます。これにより、一度に 1 つの障害に対してのみフォールト トレランスが実現されます。 これは、RAID-5 に非常に類似しています。 シングル パリティを使用するには、少なくとも 3 つのハードウェア障害ドメインが必要になります (記憶域スペース ダイレクトでは 3 台のサーバーを意味します)。 3 方向ミラーリングを使用した場合は、同じ規模でより優れたフォールト トレランスが実現されるため、シングル パリティを使用することはお勧めしません。 ただし、必要な場合はシングル パリティを使用しても問題はありません。また、完全にサポートされています。

   >[!WARNING]
   > シングル パリティが一度に安全に対処できるハードウェア障害は 1 件だけであるため、シングル パリティを使うことはお勧めしません。1 台のサーバーを再起動している場合に、別のドライブまたはサーバーで突然障害が発生した場合、ダウンタイムが発生します。 サーバーが 3 台のみの場合は、3 方向ミラーリングを使用することをお勧めします。 サーバーが 4 台以上ある場合は、次のセクションをご覧ください。

### <a name="dual-parity"></a>デュアル パリティ

デュアル パリティではリード-ソロモン エラー訂正符号が実装され、ビット単位のパリティ シンボルが 2 つ保持されます。これにより、3 方向ミラーリングと同じフォールト トレランスが実現されます (つまり、一度に最大 2 件の障害に対処できます)。また、より優れた記憶域の効率が実現されます。 これは、RAID-6 に非常に類似しています。 デュアル パリティを使用するには、少なくとも 4 つのハードウェア障害ドメインが必要になります (記憶域スペース ダイレクトでは 4 台のサーバーを意味します)。 この規模では、記憶域の効率は 50% です。つまり、2 TB のデータを保存するには、4 TB の物理的な記憶域の容量が必要になります。

![デュアル パリティ](media/Storage-Spaces-Fault-Tolerance/dual-parity-180px.png)

ハードウェア障害ドメインの数が増えると、デュアル パリティの記憶域の効率が 50% から最大で 80% まで向上します。 たとえば、ハードウェア障害ドメインが 7 つの場合 (記憶域スペース ダイレクトでは 7 台のサーバーを意味します)、効率が 66.7% に上昇します。つまり、4 TB のデータを保存するには、6 TB の物理的な記憶域の容量が必要になります。

![デュアル パリティ (規模を拡大)](media/Storage-Spaces-Fault-Tolerance/dual-parity-wide-180px.png)

各規模におけるデュアル パリティの効率とローカル再構築コードについては、「[まとめ](#summary)」セクションをご覧ください。

### <a name="local-reconstruction-codes"></a>ローカル再構築コード

Windows Server 2016 の記憶域スペースでは、Microsoft Research によって開発された高度な手法である "ローカル再構築コード" (LRC) が導入されています。 規模が大きい場合、デュアル パリティでは LRC を使用して、パリティのエンコードやデコードをいくつかの小さなグループに分割し、書き込みや障害からの回復で必要となるオーバーヘッドを削減します。

ハード ディスク ドライブ (HDD) ではグループのサイズは 4 つのシンボルで表され、ソリッド ステート ドライブ (SSD) ではグループのサイズは 6 つのシンボルで表されます。 たとえば、ハード ディスク ドライブと 12 個のハードウェア障害ドメイン (12 台のサーバー) を使用したレイアウトがどのようになるかを、以下に示します。ここでは、4 つのデータ シンボルを持つ 2 つのグループが示されています。 この場合、72.7% の記憶域の効率が実現されます。

![ローカル再構築コード](media/Storage-Spaces-Fault-Tolerance/local-reconstruction-codes-180px.png)

[ローカル再構築コードでのさまざまな障害シナリオの処理方法や、ローカル再構築コードが注目に値する理由](https://blogs.technet.microsoft.com/filecab/2016/09/06/volume-resiliency-and-efficiency-in-storage-spaces-direct/)について詳しくかつ非常にわかりやすく説明したチュートリアルをご覧になることをお勧めします。このチュートリアルは弊社の [Claus Joergensen](https://twitter.com/clausjor) によって作成されました。

## <a name="mirror-accelerated-parity"></a>ミラーリングによって高速化されたパリティ

Windows Server 2016 以降では、記憶域スペース ダイレクトの 1 つのボリュームに対して、一部ではミラーを使用し、一部ではパリティを使用することができます。 書き込みはまずミラーリングされた部分に行われ、徐々にパリティ部分に移行していきます。 実際には、これは[イレイジャー コーディングを高速化するためにミラーリングを使っています](https://blogs.technet.microsoft.com/filecab/2016/09/06/volume-resiliency-and-efficiency-in-storage-spaces-direct/)。

3 方向ミラーとデュアル パリティを混在させるには、少なくとも 4 つの障害ドメイン (4 台のサーバー) が必要になります。

ミラーリングによって高速化されたパリティの記憶域の効率は、すべてをミラーで実行した場合とすべてをパリティで実行した場合の間の値を示します。この効率は、混在させる割合によって異なります。 たとえば、[このプレゼンテーションの 37 分から始まるデモでは、12 台のサーバーに基づいて 46%、54%、65% の効率を実現するさまざまな混合の例](https://www.youtube.com/watch?v=-LK2ViRGbWs&t=36m55s)が示されています。

> [!IMPORTANT]
> ほとんどのパフォーマンスが重視されるワークロードには、ミラーリングを使用することをお勧めします。 ワークロードに応じて、パフォーマンスと容量のバランスを取る方法については詳しくは、[ボリュームの計画](plan-volumes.md#choosing-the-resiliency-type)」をご覧ください。

## <a name="summary"></a>まとめ

このセクションでは、記憶域スペース ダイレクトで利用可能な回復性の種類、それぞれの種類を使用するための最小スケール要件、それぞれの種類で許容できる障害の数、対応する記憶域の効率についてまとめています。

### <a name="resiliency-types"></a>回復性の種類

|    回復性          |    障害の許容数       |    記憶域の効率      |
|------------------------|----------------------------|----------------------------|
|    双方向ミラー      |    1                       |    50.0%                   |
|    3 方向ミラー    |    2                       |    33.3%                   |
|    デュアル パリティ         |    2                       |    50.0% ～ 80.0%           |
|    混在               |    2                       |    33.3% ～ 80.0%           |

### <a name="minimum-scale-requirements"></a>最小スケール要件

|    回復性          |    最小限必要な障害ドメイン   |
|------------------------|-------------------------------------|
|    双方向ミラー      |    2                                |
|    3 方向ミラー    |    3                                |
|    デュアル パリティ         |    4                                |
|    混在               |    4                                |

   >[!TIP]
   > [シャーシまたはラックのフォールト トレランス](../../failover-clustering/fault-domains.md)を使っている場合を除き、障害ドメインの数はサーバーの数を表します。 記憶域スペース ダイレクトの最小要件を満たしている限り、各サーバーのドライブの数は、使用できる回復性の種類に影響しません。 

### <a name="dual-parity-efficiency-for-hybrid-deployments"></a>ハイブリッド展開でのデュアル パリティの効率

次の表は、ハード ディスク ドライブ (HDD) とソリッドステート ドライブ (SSD) の両方が含まれているハイブリッド展開の各スケールでの、デュアル パリティとローカル再構築コードの記憶域効率を示しています。

|    障害ドメイン      |    レイアウト           |    効率   |
|-----------------------|---------------------|-----------------|
|    2                  |    –                |    –            |
|    3                  |    –                |    –            |
|    4                  |    RS 2+2           |    50.0%        |
|    5                  |    RS 2+2           |    50.0%        |
|    6                  |    RS 2+2           |    50.0%        |
|    7                  |    RS 4+2           |    66.7%        |
|    8                  |    RS 4+2           |    66.7%        |
|    9                  |    RS 4+2           |    66.7%        |
|    10                 |    RS 4+2           |    66.7%        |
|    11                 |    RS 4+2           |    66.7%        |
|    12                 |    LRC (8, 2, 1)    |    72.7%        |
|    13                 |    LRC (8, 2, 1)    |    72.7%        |
|    14                 |    LRC (8, 2, 1)    |    72.7%        |
|    15                 |    LRC (8, 2, 1)    |    72.7%        |
|    16                 |    LRC (8, 2, 1)    |    72.7%        |

### <a name="dual-parity-efficiency-for-all-flash-deployments"></a>オールフラッシュ展開でのデュアル パリティの効率

次の表は、ソリッドステート ドライブ (SSD) のみが含まれているオールフラッシュ展開の各スケールでの、デュアル パリティとローカル再構築コードの記憶域効率を示しています。 オールフラッシュ構成では、パリティ レイアウトで、より大きいグループ サイズを使用でき、記憶域効率の向上を実現できます。

|    障害ドメイン      |    レイアウト           |    効率   |
|-----------------------|---------------------|-----------------|
|    2                  |    –                |    –            |
|    3                  |    –                |    –            |
|    4                  |    RS 2+2           |    50.0%        |
|    5                  |    RS 2+2           |    50.0%        |
|    6                  |    RS 2+2           |    50.0%        |
|    7                  |    RS 4+2           |    66.7%        |
|    8                  |    RS 4+2           |    66.7%        |
|    9                  |    RS 6+2           |    75.0%        |
|    10                 |    RS 6+2           |    75.0%        |
|    11                 |    RS 6+2           |    75.0%        |
|    12                 |    RS 6+2           |    75.0%        |
|    13                 |    RS 6+2           |    75.0%        |
|    14                 |    RS 6+2           |    75.0%        |
|    15                 |    RS 6+2           |    75.0%        |
|    16                 |    LRC (12, 2, 1)   |    80.0        |

## <a name="examples"></a>例

サーバーが 2 台のみである場合を除き、3 方向ミラーリングとデュアル パリティの両方またはいずれかを使用することをお勧めします。これらにより、フォールト トレランスが向上するためです。 具体的には、2 つの障害ドメイン (記憶域スペース ダイレクトでは 2 台のサーバーを意味する) が同時に発生した障害の影響を受けた場合でも、すべてのデータの安全性と継続的なアクセス可能性を維持できます。

### <a name="examples-where-everything-stays-online"></a>すべてがオンラインのままである例

次の 6 つの例は、3 方向ミラーリングやデュアル パリティで対処**できる**状態を示しています。

- **1.**    1 つのドライブが失われた (キャッシュ ドライブを含む)
- **2.**    1 台のサーバーが失われた

![fault-tolerance-examples-1-and-2](media/Storage-Spaces-Fault-Tolerance/Fault-Tolerance-Example-12.png)

- **3.**    1 台のサーバーと 1 つのドライブが失われた
- **4.**    異なるサーバーの 2 つのドライブが失われた

![fault-tolerance-examples-3-and-4](media/Storage-Spaces-Fault-Tolerance/Fault-Tolerance-Example-34.png)

- **5.**    2 台を超えるドライブが失われたが、影響を受けるサーバーが最大で 2 台である
- **6.**    2 台のサーバーが失われた

![fault-tolerance-examples-5-and-6](media/Storage-Spaces-Fault-Tolerance/Fault-Tolerance-Example-56.png)

... いずれの場合も、すべてのボリュームのオンライン状態が維持されます  (クラスターがクォーラムを維持することを確認してください)。

### <a name="examples-where-everything-goes-offline"></a>すべてがオフラインになる例

有効期間中は、記憶域スペースでは任意の数の障害に対応できます。これは、各障害が発生した後で、十分な時間があれば、完全な回復状態に復元されるためです。 ただし、特定の時点で、最大で 2 つの障害ドメインが障害の影響を受けても安全です。 したがって、次に示す例は、3 方向ミラーリングやデュアル パリティでは対処**できない**状態です。

- **7.** 同時に 3 台以上のサーバーでドライブが失われた
- **8.** 同時に 3 台以上のサーバーが失われた

![fault-tolerance-examples-7-and-8](media/Storage-Spaces-Fault-Tolerance/Fault-Tolerance-Example-78.png)

## <a name="usage"></a>使用方法

「[記憶域スペース ダイレクトのボリュームの作成](create-volumes.md)」をご覧ください。

## <a name="see-also"></a>関連項目

以下のすべてのリンクは、このトピックの本文内に記載されています。

- [Storage Spaces Direct in Windows Server 2016 (Windows Server 2016 の記憶域スペース ダイレクト)](storage-spaces-direct-overview.md)
- [Fault Domain Awareness in Windows Server 2016 (Windows Server 2016 での障害ドメインの認識)](../../failover-clustering/fault-domains.md)
- [Erasure Coding in Azure by Microsoft Research (Microsoft Research による Azure でのイレイジャー コーディング)](https://www.microsoft.com/en-us/research/publication/erasure-coding-in-windows-azure-storage/)
- [Local Reconstruction Codes and Accelerating Parity Volumes (ローカル再構築コードとパリティ ボリュームの高速化)](https://blogs.technet.microsoft.com/filecab/2016/09/06/volume-resiliency-and-efficiency-in-storage-spaces-direct/)
- [Volumes in the Storage Management API (Storage Management API でのボリューム)](https://blogs.technet.microsoft.com/filecab/2016/08/29/deep-dive-volumes-in-spaces-direct/)
- [Storage Efficiency Demo at Microsoft Ignite 2016 (Microsoft Ignite 2016 での記憶域の効率に関するデモ)](https://www.youtube.com/watch?v=-LK2ViRGbWs&t=36m55s)
- [Capacity Calculator for Storage Spaces Direct (記憶域スペース ダイレクトの容量計算 (プレビュー版))](http://aka.ms/s2dcalc)