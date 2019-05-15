---
title: WSUS クライアント コンピューターと WSUS コンピューター グループを管理する
description: Windows Server Update Service (WSUS) のトピックでは、クライアント コンピューターおよびグループを管理する方法
ms.custom: na
ms.prod: windows-server-threshold
ms.reviewer: na
ms.suite: na
ms.technology: manage-wsus
ms.tgt_pltfrm: na
ms.topic: article
ms.assetid: 4b1ea915-0f9f-4f0e-8913-a1dd460d07ab
author: coreyp-at-msft
ms.author: coreyp
manager: lizapo
ms.date: 10/16/2017
ms.openlocfilehash: e63aa5d53f01fbff3029c6896556d92c9c99aa50
ms.sourcegitcommit: 0d0b32c8986ba7db9536e0b8648d4ddf9b03e452
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/17/2019
ms.locfileid: "59857683"
---
# <a name="managing-wsus-client-computers-and-wsus-computer-groups"></a>WSUS クライアント コンピューターと WSUS コンピューター グループを管理する

>適用先:Windows Server (半期チャネル)、Windows Server 2016、Windows Server 2012 R2、Windows Server 2012

コンピューター ノードは、WSUS クライアント コンピューターとデバイスを管理するためには、WSUS 管理コンソールで集約型アクセス ポイントです。 このノードの下、別のグループを設定する (および、既定のグループ割り当てられていないコンピューター) を見つけることができます。

## <a name="managing-client-computers"></a>クライアント コンピュータの管理
内のコンピューター グループのいずれかを選択、**コンピューター**ノードの下**オプション**詳細ウィンドウに表示するには、そのグループで、コンピューターが発生します。 コンピューターが複数のグループに割り当てられた両方のグループの一覧に表示されます。 場合、 一覧で、コンピューターを選択すると、そのプロパティは、コンピューターとインストールなど、更新プログラムの状態または特定のコンピューターの更新プログラムの検出状態に関する一般的な詳細情報が含まれますが確認できます。 状態では、特定のコンピューター グループに属するコンピューターの一覧をフィルター処理できます。 既定値は、必要な更新またはインストールの失敗がされていたコンピューターにのみを示しています。ただし、すべての状態で表示をフィルター処理できます。 クリックして**更新**ステータス フィルタを変更した後。

[コンピューター] ページで、グループを作成し、それらへのコンピューターの割り当てを含むコンピュータ グループを管理することもできます。 コンピューター グループの管理に関する詳細については、[次へ] のこのガイドのセクションとセクションのコンピューター グループの管理を参照してください[1.5。WSUS コンピューター グループを計画](../plan/plan-your-wsus-deployment.md#BKMK_1.5)手順 1。WSUS 展開ガイドの WSUS 展開を準備します。

> [!NOTE]
> 最初に、そのサーバーから管理するには、WSUS サーバーをアクセス クライアント コンピューターを構成する必要があります。 このタスクを実行するまで、WSUS サーバーは、クライアント コンピューターを認識しないと、[コンピューター] ページの一覧に表示されません。 クライアント コンピューターの設定に関する詳細については、次を参照してください。 [1.5。WSUS コンピューター グループを計画](../plan/plan-your-wsus-deployment.md#BKMK_1.5)の手順 1。WSUS の展開を準備し、手順 3。WSUS 展開ガイドでは、WSUS を構成します。

## <a name="controlling-when-wsus-client-computers-install-updates"></a>WSUS クライアント コンピューターが更新プログラムをインストールするときに制御します。
WSUS クライアント コンピューターの更新プログラムをインストールするときに、コントロールに 2 つの方法があります。

-   期限の承認:更新プログラムのインストール時に期限が厳密に適用します。

-   WSUS グループ ポリシー:グループ ポリシー、Windows Update エージェントをスキャンし、更新プログラムをインストールするときの制御します。

    詳細についてを参照してください。[手順 5:自動更新のグループ ポリシー設定を構成する](../deploy/4-configure-group-policy-settings-for-automatic-updates.md)、WSUS Deployment guide の「します。

## <a name="managing-computer-groups"></a>コンピューター グループの管理
WSUS では、クライアント コンピューターのグループを更新プログラム展開の対象とすることができるため、特定のコンピューターは常に最も適切な時間に適切な更新プログラムを取得できます。 たとえば、ある部署 (経理チーム) 内のすべてのコンピューターが特定の構成になっている場合、そのチームのグループを設定し、コンピューターで必要な更新プログラムと更新プログラムをインストールするタイミングを決定し、WSUS レポートを使用してチームの更新プログラムを評価できます。

コンピューターが常に割り当てられている、**すべてのコンピューター**グループ化、およびに割り当てられたまま、**コンピューターが割り当てられていない**グループを別のグループに割り当てるまでです。 コンピューターは複数のグループに所属することができます。

コンピューター グループは階層化できます (たとえば、経理用の Accounting グループの下に給与関連の Payroll グループと支払勘定関連の Accounts Payable グループを設定できます)。 上位グループが承認された更新プログラムは、上位グループ自体のほか、下位のグループに自動的にデプロイされます。 したがって、"経理"グループの更新プログラム 1 を承認する場合、更新プログラムは経理グループのすべてのコンピューター、すべてのコンピューター、Payroll グループと Accounts Payable グループのすべてのコンピューターに配置されます。

コンピューターは複数のグループに割り当てることができるため、同じコンピューターで 1 つの更新プログラムを何度も承認することができます。 ただし、その更新プログラムが展開されるのは一度だけです。競合が発生した場合は WSUS サーバーで解決されます。 コンピューター a は、給与と Accounts Payable グループの両方に割り当てられ、更新プログラム 1 が両方のグループに対して承認された場合は、上記の例を続行に配置される 1 回だけです。

コンピューター グループにコンピューターを割り当てる方法は 2 つあり、サーバー側でのターゲット指定とクライアント側でのターゲット指定が可能です。 サーバー側でターゲット指定する、手動で移動する 1 つまたは複数のクライアント コンピューター 1 台のコンピューター グループに、時にします。 クライアント側を対象とする、グループ ポリシーを使用するか自動的に以前に作成したコンピューター グループに追加するには、そのコンピューターを有効にするクライアント コンピューターでレジストリ設定を編集します。 このプロセスをスクリプト化され、一度に多数のコンピューターに展開することができます。 2 つのオプションのいずれかを選択して、WSUS サーバーで使用するターゲットのメソッドを指定する必要があります、**コンピューター**のセクション、**オプション**ページ。

> [!NOTE]
> WSUS サーバーがレプリカ モードで実行されている場合、そのサーバーにコンピューター グループを作成できません。 WSUS サーバー階層のルートである WSUS サーバーでレプリカ サーバーのクライアントに必要なすべてのコンピューター グループを作成する必要があります。 レプリカ モードの詳細については、次を参照してください。[を実行している WSUS レプリカ モード](running-wsus-replica-mode.md)サーバー側とクライアント側のターゲット設定についての詳細については、セクションを参照してください。 [1.5。WSUS コンピューター グループを計画](../plan/plan-your-wsus-deployment.md#BKMK_1.5)の手順 1。WSUS 展開ガイドでは、WSUS 展開を準備します。

