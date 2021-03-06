---
title: Windows 認証で使用されるグループ ポリシー設定
description: Windows Server のセキュリティ
ms.custom: na
ms.prod: windows-server
ms.reviewer: na
ms.suite: na
ms.technology: security-windows-auth
ms.tgt_pltfrm: na
ms.topic: article
ms.assetid: 9e237f89-45b1-4a4e-9b72-11dc7d6a470b
author: coreyp-at-msft
ms.author: coreyp
manager: dongill
ms.date: 10/12/2016
ms.openlocfilehash: 352e67dead6e22085a8e7350dd7e5c18c6d74676
ms.sourcegitcommit: 6aff3d88ff22ea141a6ea6572a5ad8dd6321f199
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/27/2019
ms.locfileid: "71403253"
---
# <a name="group-policy-settings-used-in-windows-authentication"></a>Windows 認証で使用されるグループ ポリシー設定

>適用先:Windows Server (半期チャネル)、Windows Server 2016

IT プロフェッショナル向けのこのリファレンストピックでは、認証プロセスにおけるグループポリシー設定の使用方法と影響について説明します。

Windows オペレーティングシステムで認証を管理するには、ユーザー、コンピューター、およびサービスアカウントをグループに追加してから、それらのグループに認証ポリシーを適用します。 これらのポリシーは、ローカルセキュリティポリシーとして定義され、管理用テンプレート (グループポリシー設定とも呼ばれます) として定義されます。 両方のセットは、グループポリシーを使用して、組織全体で構成および配布できます。

> [!NOTE]
> Windows Server 2012 R2 で導入された機能を使用すると、保護されたアカウントを使用して、対象となるサービスまたはアプリケーション (一般に認証サイロと呼ばれます) の認証ポリシーを構成できます。 Active Directory でこれを行う方法については、「[保護されたアカウントの構成方法](how-to-configure-protected-accounts.md)」を参照してください。

たとえば、組織内の機能に基づいて、次のポリシーをグループに適用することができます。

-   ローカルまたはドメインへのログオン

-   ネットワーク経由でログオンする

-   アカウントのリセット

-   アカウントの作成

次の表に、認証に関連するポリシーグループと、それらのポリシーを構成するのに役立つドキュメントへのリンクを示します。

|ポリシーグループ|Location|説明|
|--------|------|--------|
|**パスワードポリシー**|Local Computer ポリシー \ の \Windows \ システムポリシー|パスワードポリシーは、パスワードの特性と動作に影響します。 パスワードポリシーは、ドメインアカウントまたはローカルユーザーアカウントに使用されます。 これらは、パスワードの設定 (適用や有効期間など) を決定します。<br /><br />特定の設定の詳細については、「[パスワードポリシー](https://technet.microsoft.com/itpro/windows/keep-secure/password-policy)」を参照してください。|
|**アカウントロックアウトのポリシー**|Local Computer ポリシー \ の \Windows \ システムポリシー|アカウントロックアウトのポリシーオプションでは、ログオンに失敗した回数を設定した後、アカウントを無効にします。 これらのオプションを使用すると、パスワードを中断する試行を検出してブロックするのに役立ちます。<br /><br />アカウントロックアウトのポリシーオプションの詳細については、「[アカウントロックアウトポリシー](https://technet.microsoft.com/itpro/windows/keep-secure/account-lockout-policy)」を参照してください。|
|**Kerberos ポリシー**|Local Computer ポリシー \ の \Windows \ システムポリシー|Kerberos 関連の設定には、チケットの有効期間と適用規則が含まれます。 Kerberos 認証プロトコルはローカルアカウントの認証には使用されないため、kerberos ポリシーはローカルアカウントデータベースには適用されません。 したがって、Kerberos ポリシー設定を構成できるのは、既定のドメイングループポリシーオブジェクト (GPO) を使用する方法のみです。この場合、ドメインログオンに影響します。<br /><br />ドメインコントローラーの Kerberos ポリシーオプションの詳細については、「 [Kerberos ポリシー](https://technet.microsoft.com/itpro/windows/keep-secure/kerberos-policy)」を参照してください。|
|**監査ポリシー**|Local Computer ポリシー \ \Windows \ 権利 Policies\Audit Policy|監査ポリシーを使用すると、ファイルやフォルダーなどのオブジェクトへのアクセスを制御および理解したり、ユーザーアカウントやグループアカウント、ユーザーのログオンとログオフを管理したりすることができます。 監査ポリシーでは、監査するイベントのカテゴリを指定したり、セキュリティログのサイズと動作を設定したり、アクセスを監視するオブジェクトと監視するアクセスの種類を決定したりできます。<br /><br />|
|**ユーザー権利の割り当て**|Local Computer ポリシー \ \Windows \ 権利権限の割り当て|ユーザー権利は、通常、ユーザーが属するセキュリティグループ (管理者、パワーユーザー、ユーザーなど) に基づいて割り当てられます。 このカテゴリのポリシー設定は、通常、アクセス許可とセキュリティグループのメンバーシップに基づいて、コンピューターにアクセスするためのアクセス許可を付与または拒否するために使用されます。|
|**セキュリティオプション**|Local Computer ポリシー \ \Windows \ 権利セキュリティ Options|認証に関連するポリシーには次のものがあります。<br /><br />-デバイス<br />-ドメインコントローラー<br />-ドメインメンバー<br />-対話型ログオン<br />-Microsoft ネットワークサーバー<br />-ネットワークアクセス<br />-ネットワークセキュリティ<br />-回復コンソール<br />-Shutdown<br /><br />|
|**資格情報の委任**|コンピューターの構成 \ 管理用テンプレート \ システム \ 資格情報の委任|資格情報の委任は、他のシステム、特にドメイン内のメンバーサーバーとドメインコントローラーでローカルの資格情報を使用できるようにするメカニズムです。 これらの設定は、Credential Security Support Provider (Cred SSP) を使用してアプリケーションに適用されます。 リモートデスクトップ接続の例を次に示します。|
|**KDC**|コンピューターの構成 \ 管理用テンプレート \ システムの KDC|これらのポリシー設定は、ドメインコントローラー上のサービスであるキー配布センター (KDC) が Kerberos 認証要求を処理する方法に影響します。|
|**満たさ**|コンピューターの構成 \ 管理用テンプレート \ システム-Kerberos|これらのポリシー設定は、信頼性情報、Kerberos 防御、複合認証、プロキシサーバーの識別、およびその他の構成のサポートを処理するための Kerberos の構成方法に影響します。|
|**ログオン**|コンピューターの構成\管理用テンプレート\システム\ログオン|これらのポリシー設定は、ユーザーのログオンエクスペリエンスをシステムがどのように表示するかを制御します。|
|**Net ログオン**|コンピューターの構成 \ 管理用テンプレート \ システム \ ログオン|これらのポリシー設定は、ドメインコントローラーロケーターの動作など、システムがネットワークログオン要求を処理する方法を制御します。<br /><br />ドメインコントローラーロケーターがレプリケーションプロセスにどのように適合するかの詳細については、「[サイト間のレプリケーション](https://technet.microsoft.com/library/cc771251.aspx)について」を参照してください。|
|**生体認証**|コンピューターの構成 \ 管理用テンプレート \Windows コンポーネント Components\Biometrics|これらのポリシー設定は、通常、認証方法として生体認証の使用を許可または拒否します。<br /><br />生体認証の Windows 実装の詳細については、「Windows 生体認証フレームワークの概要」を参照してください。|
|**資格情報のユーザーインターフェイス**|コンピューターの構成 \ 管理用テンプレート \Windows コンポーネント \Windows ユーザーインターフェイス|これらのポリシー設定は、エントリの時点で資格情報を管理する方法を制御します。|
|**パスワード同期**|コンピューターの構成 \ 管理用テンプレート \Windows コンポーネント Components\Password 同期|これらのポリシー設定は、Windows と UNIX ベースのオペレーティングシステム間でのパスワードの同期をシステムがどのように管理するかを決定します。<br /><br />詳細については、「[パスワード同期](https://technet.microsoft.com/library/cc732609.aspx)」を参照してください。|
|**スマートカード**|コンピューターの構成 \ 管理用テンプレート \Windows コンポーネント Components\Smart カード|これらのポリシー設定は、システムがスマートカードログオンを管理する方法を制御します。<br /><br />|
|**Windows のログオンオプション**|コンピューターの構成 \ 管理用テンプレート \Windows コンポーネント \Windows ログオンオプション|これらのポリシー設定は、ログオンの営業案件をいつどのように利用できるかを制御します。|
|**Ctrl + Alt + Del オプション**|コンピューターの構成 \ 管理用テンプレート \Windows コンポーネント Components\Ctrl + Alt + Del オプション|これらのポリシー設定は、ログオン UI (セキュリティで保護されたデスクトップ) の機能 (タスクマネージャーやコンピューターのキーボードロックなど) の外観とアクセシビリティに影響します。|
|**ログオン**|コンピューターの構成 \ 管理用テンプレート \Windows コンポーネント Components\Logon|これらのポリシー設定は、ユーザーがログオンしたときに実行できるかどうかを決定します。|

## <a name="see-also"></a>関連項目
[Windows 認証の技術概要](windows-authentication-technical-overview.md)


