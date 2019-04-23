---
title: セット id
description: 'Windows コマンド」のトピック * * *- '
ms.custom: na
ms.prod: windows-server-threshold
ms.reviewer: na
ms.suite: na
ms.technology: manage-windows-commands
ms.tgt_pltfrm: na
ms.topic: article
ms.assetid: 5793d7ad-827e-4285-b2c6-ae60eeb0e886
author: coreyp-at-msft
ms.author: coreyp
manager: dongill
ms.date: 10/16/2017
ms.openlocfilehash: f95490850acd263fb0b34007ac64a84c9a374865
ms.sourcegitcommit: 0d0b32c8986ba7db9536e0b8648d4ddf9b03e452
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/17/2019
ms.locfileid: "59822053"
---
# <a name="set-id"></a>セット id

>適用先:Windows Server (半期チャネル)、Windows Server 2016、Windows Server 2012 R2、Windows Server 2012

系列 ID の Diskpart コマンドでは、フォーカスのあるパーティションのパーティションの種類フィールドを変更します。  
  
> [!IMPORTANT]  
> このコマンドは、供給元で使用するため \(Oem\) のみです。 このパラメーターを持つパーティションの種類フィールドを変更すると、エラーが発生したり、起動できなくなる、コンピューターが発生する可能性があります。 Oem または gpt ディスクの使用が発生すると、しない限り、このパラメーターを使用して gpt ディスク上のパーティションの種類フィールドを変更する必要がありますできません。 代わりに、常に使用して、 [efi のパーティションを作成](create-partition-efi.md)EFI システム パーティションを作成するコマンド、[パーティション msr を作成](create-partition-msr.md)Microsoft 予約パーティションを作成するコマンドと[作成プライマリ パーティション](create-partition-primary.md)gpt ディスク上のプライマリ パーティションを作成する ID パラメーターを指定しないコマンド。  
  
  
  
## <a name="syntax"></a>構文  
  
```  
set id={ <byte> | <GUID> } [override] [noerr]  
```  
  
## <a name="parameters"></a>パラメーター  
  
|パラメーター|説明|  
|-------|--------|  
|<byte>|マスター ブート レコードの\(MBR\)ディスク、パーティション用の 16 進形式の種類 フィールドに新しい値を指定します。 LDM パーティションを指定する型 0x42 を除き、このパラメーターでは、すべてのパーティションの種類のバイトを指定できます。 16 進数のパーティションの種類を指定するときに、先頭の 0x を省略したことに注意してください。|  
|<GUID>|GUID パーティション テーブルの\(gpt\)ディスク、パーティションの種類 フィールドに新しい GUID 値を指定します。 認識された Guid は次のとおりです。<br /><br />-EFI システム パーティション: c12a7328\-f81f\-11 d 2\-ba4b\-00a0c93ec93b<br />-ベーシック データ パーティション: ebd0a0a2\-b9e5\-4433\-87 c 0\-68b6b72699c7<br /><br />ただし、次は、このパラメーターでは、パーティションの種類の GUID を指定できます。<br /><br />Microsoft 予約パーティション: e3c9e316\-0b5c\-4db8\-817 d\-f92df00215ae<br />のダイナミック ディスク: 5808c8aa 上 LDM メタデータ パーティション\-7e8f\-42e0\-85 d 2\-e1e90434cfb3<br />ダイナミック ディスク上の LDM データ パーティション: af9b60a0\-1431\-4f62\-bc68\-3311714a69ad<br />-クラスターのメタデータ パーティション: db97dba9\-0840\-4bae\-97f0\-ffb9a327c7e1|  
|上書き|パーティションの種類を変更する前にマウントを解除するボリューム上のファイル システムを強制します。 実行すると、 **id** コマンドを DiskPart はロックし、ボリューム上のファイル システムをマウント解除を試みます。 場合 **オーバーライド** が指定されていないファイル システムをロックする呼び出しが失敗して \(などの開いているハンドルがあるため\), 、操作は失敗します。 **オーバーライド** が指定されている場合でも、ファイル システムのロックへの呼び出しが失敗すると、および、ボリュームに開いているハンドルが無効になり、DiskPart がマウントが解除を強制します。<br /><br />このコマンドでは、Windows 7 および Windows Server 2008 R2 の使用のみです。|  
|noerr|スクリプトのみに使用されます。 エラーが発生すると、DiskPart は、エラーが発生しなかったかのようにコマンドを処理し続けます。 このパラメーターは、エラー発生すると、DiskPart はエラー コードを生成して終了します。|  
  
## <a name="remarks"></a>注釈  
  
-   前述の制限、以外 DiskPart 調べませんを指定する値の有効性 \(することを確認バイトの 16 進形式または GUID を除く\)します。  
  
-   このコマンドは、ダイナミック ディスク上または Microsoft 予約パーティション上には機能しません。  
  
## <a name="BKMK_examples"></a>例  
型のフィールドに 0x07 設定を強制的にマウントを解除するファイル システムは、次のように入力します。  
  
```  
set id=0x07 override  
```  
  
ベーシック データ パーティションにする型のフィールドを設定するには、次のように入力します。  
  
```  
set id=ebd0a0a2-b9e5-4433-87c0-68b6b72699c7  
```  
  
#### <a name="additional-references"></a>その他の参照  
[コマンドライン構文キー](command-line-syntax-key.md)  
  

  
