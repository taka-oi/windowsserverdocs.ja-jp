---
title: Windows Server Essentials の移行先サーバーでフォルダー リダイレクトを構成する
description: Windows Server Essentials を使用する方法について説明します
ms.custom: na
ms.date: 10/03/2016
ms.prod: windows-server-2016-essentials
ms.reviewer: na
ms.suite: na
ms.tgt_pltfrm: na
ms.topic: article
ms.assetid: fe77ba67-128c-4fc3-9361-30fa6af42516
author: nnamuhcs
ms.author: coreyp
manager: dongill
ms.openlocfilehash: 6709aa2a342efd5837cf340b645961499baff6a5
ms.sourcegitcommit: eaf071249b6eb6b1a758b38579a2d87710abfb54
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/31/2019
ms.locfileid: "66433012"
---
# <a name="configure-folder-redirection-on-the-windows-server-essentials-destination-server"></a>Windows Server Essentials の移行先サーバーでフォルダー リダイレクトを構成する

>適用先:Windows Server 2016 Essentials、Windows Server 2012 R2 Essentials、Windows Server 2012 Essentials

移行元サーバーでフォルダー リダイレクトが有効になっている場合、このタスクを実行します。  
  
 最初に、古いフォルダー リダイレクト グループ ポリシーの設定を削除します。 Windows Server Essentials ダッシュ ボードを使用して、移行先サーバーでフォルダー リダイレクトを有効にします。  
  
### <a name="to-delete-the-old-folder-redirection-group-policy-setting"></a>古いフォルダー リダイレクト グループ ポリシーの設定を削除するには  
  
1. 移行先サーバーで、 **[グループ ポリシーの管理]** 管理ツールを開きます。  
  
2. **Group Policy Management**、展開**フォレスト:** <em>ネットワーク ドメイン名</em>、展開**ドメイン**、展開*ネットワーク ドメイン名*、順に展開**グループ ポリシー オブジェクト**します。  
  
3. **[SBS グループ ポリシー フォルダー リダイレクト]** を右クリックし、 **[削除]** をクリックします。  
  
4. **[SBS グループ ポリシー セキュリティ テンプレート]** を右クリックして、 **[削除]** をクリックします。  
  
5. 警告を読み、 **[はい]** をクリックします。  
  
6. **[グループ ポリシーの管理]** を閉じます。  
  
### <a name="to-enable-folder-redirection-on-the-destination-server"></a>移行先サーバーでフォルダー リダイレクトを有効にするには  
  
1. 移行先サーバーでは、Windows Server Essentials ダッシュ ボードを開きます。  
  
2. ナビゲーション バーで、 **[デバイス]** をクリックします。  
  
3. **[デバイスのタスク]** ウィンドウで、 **[グループ ポリシーの実装]** をクリックします。  
  
4. **[フォルダー リダイレクト グループ ポリシーを有効にする]** ページで、リダイレクトするフォルダーを選択し、 **[次へ]** をクリックします。  
  
5. **[セキュリティ ポリシー設定を有効にする]** ページで、 **[完了]** をクリックします。  
  
   フォルダー リダイレクトの変更を適用するには、ネットワーク ユーザーはコンピューターからログオフした後、ログオンし直す必要があります。 これにより、リダイレクトされるすべてのフォルダーが移行先サーバーに転送されます。
