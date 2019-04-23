---
ms.assetid: e34622ff-b2d0-4f81-8d00-dacd5d6c215e
title: Windows タイム サービスのテクニカル リファレンス
description: ''
author: shortpatti
ms.author: pashort
manager: brianlic
ms.date: 02/01/2018
ms.topic: article
ms.prod: windows-server-threshold
ms.technology: networking
ms.openlocfilehash: 916ac654aeb1ca56e0283f9a18e011ef67734776
ms.sourcegitcommit: 19d9da87d87c9eefbca7a3443d2b1df486b0b010
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/28/2018
---
# <a name="windows-time-service-technical-reference"></a>Windows タイム サービスのテクニカル リファレンス
W32Time サービスは、ネットワークの広範な構成なしのコンピューターの時計の同期を提供します。 W32Time サービスは、Kerberos V5 認証の操作が成功して、そのため、AD DS ベースの認証に不可欠です。 ほとんどのセキュリティ サービスを含む、任意の Kerberos に対応したアプリケーションは、認証要求に参加しているコンピューター間で時刻の同期に依存します。 AD DS ドメイン コントローラーも正確なデータのレプリケーションを確認するためにクロックを同期している必要があります。

> [!NOTE]  
> Windows Server 2003 および Microsoft Windows 2000 Server で、ディレクトリ サービスは Active Directory ディレクトリ サービスを名前します。 Windows Server 2008 R2 および Windows Server 2008 では、ディレクトリ サービスは Active Directory ドメイン サービス (AD DS) の名前します。 このトピックの残りの部分は、AD DS を指しますが、情報も Windows Server 2016 での Active Directory ドメイン サービスに適用可能です。

W32Time サービスは W32Time.dll、既定でインストールされると呼ばれるダイナミック リンク ライブラリで実装**%Systemroot%\System32**します。 W32Time.dll は、時計を同期するためのネットワーク上の必要な Kerberos V5 認証プロトコルでの仕様をサポートするために Windows 2000 Server のもともと開発されました。 Windows Server 2003 以降では、W32Time.dll は、Windows Server 2000 オペレーティング システム経由でネットワーク時計の同期の精度が向上を提供します。 さらに、Windows Server 2003、W32Time.dll は、さまざまなハードウェア デバイスとタイム プロバイダーを使用して、ネットワーク タイム プロトコルをサポートします。

Kerberos 認証の時計の同期を提供するように設計されて、多くの現在のアプリケーションを使用してタイムスタンプするトランザクションの一貫性を確保する、重要なイベント、およびその他のビジネス クリティカルな時刻を記録時間に依存情報です。  これらのアプリケーション、Windows タイム サービスによって提供されるコンピューター間で時刻の同期を享受できます。

## <a name="importance-of-time-protocols"></a>時間プロトコルの重要性
時間プロトコルは、時刻情報を交換し、その情報を使用してその時計を同期する 2 台のコンピューター間で通信します。 Windows タイム サービス時間プロトコルでは、クライアントは、サーバーから時刻の情報を要求し、受信する情報に基づくその時計を同期させます。
  
Windows タイム サービスは、NTP を使用して、ネットワーク経由で時間を同期します。 NTP は、時計を同期させるために必要な作業分野のアルゴリズムを含むインターネット時刻プロトコルです。 NTP がより正確なタイム プロトコルよりも、単純なネットワーク タイム プロトコル (SNTP) Windows; の一部のバージョンで使用されています。ただし、W32Time は、Windows 2000 などの SNTP ベースのタイム サービスを実行しているコンピューターとの下位互換性を有効にする SNTP をサポートするためには続行されます。
<!-- maybe this should be its own topic under the Tech Ref section -->
## <a name="BKMK_Config"></a>Windows タイム サービスの構成に関する情報を検索する場所  
このガイドは**いない**Windows タイム サービスの構成について説明します。 いくつか異なるトピックは Microsoft technet の「と、Microsoft サポート技術情報では、Windows タイム サービスを構成するため手順を説明します。 構成情報を必要とする場合、次のトピックは、適切な情報を見つけるに役立ちます。  
<!-- should this be an if/then table -->
-   フォレスト ルート プライマリ ドメイン コントローラー (PDC) エミュレーターの Windows タイム サービスを構成するのには、次を参照してください。  
  
    -   [フォレスト ルート ドメインの PDC エミュレーターで Windows タイム サービスを構成します。](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc731191%28v=ws.10%29) 
  
    -   [フォレストのタイム ソースを構成します。](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-r2-and-2008/cc794823%28v%3dws.10%29) 
  
    -   Microsoft サポート技術情報の記事 816042、[を Windows Server では、信頼できるタイム サーバーを構成する方法](https://go.microsoft.com/fwlink/?LinkID=60402)、Windows Server 2008 R2、Windows Server 2008、Windows Server 2003 を実行しているコンピューターの構成設定について説明します。および Windows Server 2003 R2 します。  
  
-   ドメイン メンバー クライアントまたはサーバー、またはフォレスト ルート PDC エミュレーターとして構成されていないするものドメイン コントローラーには、Windows タイム サービスを構成するを参照してください。[自動ドメイン時刻の同期用のクライアント コンピューターを構成する](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-r2-and-2008/cc816884%28v%3dws.10%29)します。  
  
    > [!WARNING]  
    > 一部のアプリケーションでは、自分のコンピューターが正確度の高いタイム サービスを必要があります。 場合は、Windows タイム サービスが、正確なタイム ソースとして機能する設計しないことに注意してください、手動のタイム ソースを構成し、ことができます。 Microsoft サポート技術情報の記事 939322、」の説明に従って正確度の高いタイム環境のサポートの制約を認識していることを確認[高精度環境向けの Windows タイム サービスを構成するサポート境界](https://go.microsoft.com/fwlink/?LinkID=179459)します。  
  
-   どの Windows ベース クライアントまたはサーバー コンピューターにドメインのメンバーではなくワークグループのメンバーのように構成されている Windows タイム サービスを構成する[選択したクライアント コンピューターの手動のタイム ソースを構成する](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-r2-and-2008/cc816656%28v%3dws.10%29)します。  
  
-   Windows タイム サービスを構成する仮想環境を実行するホスト コンピューターで、Microsoft サポート技術情報の記事 816042 を参照してください。[を Windows Server では、信頼できるタイム サーバーを構成する方法](https://go.microsoft.com/fwlink/?LinkID=60402)します。 Microsoft 以外の仮想化製品で作業している場合にして、その製品のベンダーのドキュメントを参照してください。  
  
-   仮想マシンで実行されているドメイン コントローラーで Windows タイム サービスを構成するには、ドメイン コントローラーとして動作しているホスト システムとゲスト オペレーティング システムの間で時刻の同期が部分的に無効にすることをお勧めします。 これにより、ドメインの階層の時刻を同期するドメイン コントローラー ゲストが、時刻のずれが復元した場合は、保存した状態からの発生から保護します。 詳細については、Microsoft サポート技術情報の記事 976924 を参照してください[Hyper-v と Windows Server 2008 ベースのホスト サーバーで実行されている仮想化ドメイン コントローラー上の Windows タイム サービス イベント ID 24、29、および 38 を受信する](https://go.microsoft.com/fwlink/?LinkID=192236)と。[仮想化ドメイン コントローラーの展開に関する考慮事項](https://go.microsoft.com/fwlink/?LinkID=192235)します。  
  
-   仮想コンピューターで実行されても、フォレスト ルート PDC エミュレーターとして機能するドメイン コントローラーで Windows タイム サービスを構成する手順は同じ物理コンピューターの」の説明に従って[PDC 上で Windows タイム サービスを構成します。フォレスト ルート ドメインのエミュレーター](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc731191%28v=ws.10%29)します。  
  
-   仮想コンピューターとして実行されているメンバー サーバー上の Windows タイム サービスを構成するを使用して、ドメインに階層」の説明に従って ([自動ドメイン時刻の同期用のクライアント コンピューターを構成する](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-r2-and-2008/cc816884%28v%3dws.10%29)します。


> [!IMPORTANT]  
> Windows Server 2016 では、前に W32Time サービスしない時間の影響を受けやすいアプリケーションのニーズに合わせて設計されました。  ただし、Windows Server 2016 に更新できるようになりましたミリ秒のソリューションを実装する、ドメイン内の正確性。  See [Windows 2016 Accurate Time](accurate-time.md) and  [Support boundary to configure the Windows Time service for high-accuracy environments](https://go.microsoft.com/fwlink/?LinkID=179459) for more information.

## <a name="related-topics"></a>関連するトピック  
[Windows タイム サービスのしくみ](How-the-Windows-Time-Service-Works.md)  
[Windows タイム サービスのツールと設定](Windows-Time-Service-Tools-and-Settings.md)  
[Microsoft サポート技術情報の記事 902229](https://go.microsoft.com/fwlink/?LinkId=186066)