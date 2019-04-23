---
title: ドライブ文字を変更します。
description: ディスクの管理を使用して Windows でドライブ文字の割り当てを変更する方法。
ms.date: 10/24/2018
ms.prod: windows-server-threshold
ms.technology: storage
ms.topic: article
author: JasonGerend
manager: brianlic
ms.author: jgerend
ms.openlocfilehash: 5f25d49ac399633c048b0c8581551d862145ca76
ms.sourcegitcommit: 0d0b32c8986ba7db9536e0b8648d4ddf9b03e452
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/17/2019
ms.locfileid: "59812163"
---
# <a name="change-a-drive-letter"></a>ドライブ文字を変更します。

> **適用対象します。** Windows 10、Windows 8.1、Windows 7、Windows Server (半期チャネル)、Windows Server 2016、Windows Server 2012 R2、Windows Server 2012

ドライブに割り当てられたドライブ文字に満足できない、またはドライブ文字がまだでするドライブがある場合、ディスク管理を使用してこれを変更することができます。

> [!IMPORTANT]
> Windows またはアプリがインストールされているドライブのドライブ文字を変更する場合、アプリを実行している、またはそのドライブの検索に問題がある可能性があります。 このため、Windows またはアプリがインストールされているドライブのドライブ文字は変更しないことをお勧めします。

ドライブ文字を変更する方法を次に示します (代わりに、空のドライブをマウントするフォルダーとしてもう 1 つのフォルダーに表示されるようにを参照してください[マウント ポイント フォルダー パスをドライブに割り当てる](assign-a-mount-point-folder-path-to-a-drive.md))。

1. 管理者のアクセス許可を持つディスクの管理を開きます。 <br>タスク バーの検索ボックスに、これには、次のように入力します**ディスクの管理**、を選択し、保持 (または右クリック)**ディスクの管理**を選択し、**管理者として実行** > 。**はい**します。 入力の場合は、管理者として開くにはできません、**コンピュータの管理**代わりに、順に移動**ストレージ** > **ディスクの管理**します。
1. ディスク管理で、変更、またはドライブ文字を追加し、選択するドライブを右クリックして**を変更するドライブ文字とパス**します。<br>
![ドライブを表示するディスクの管理](media/change-drive-letter.png)
    > [!TIP]
    > 表示されない場合、**変更ドライブ文字とパス**またはオプションが淡色表示になる、ボリュームにドライブが割り当てられていないとする必要がある場合は、大文字と小文字を指定できるドライブ文字を受信する準備ができていないことのできる[の初期化](initialize-new-disks.md). または、おそらくではありませんアクセスするには、EFI システム パーティションと回復パーティションの場合します。 アクセス可能なドライブ文字の書式設定されたボリュームがあり、そのまま変更できないを確認したら、残念ながらこのトピックで可能性があります支援はできません、ためお勧めしますに[Microsoft に連絡](https://support.microsoft.com/contactus/)またはの製造元、詳細については PC。

1. ドライブ文字を変更するには、選択**変更**します。 ドライブを持っていない場合、1 つ選択してをドライブ文字を追加する**追加**します。<br>![変更のドライブ文字とパスのダイアログ](media/change-drive-letter2.png)
3. 新しいドライブ文字を選択して、 **ok**、し、**はい**どのドライブ文字に依存するプログラムが正しく動作しない可能性がありますの入力を求められたらします。<br>![ドライブ文字の変更を表示する変更のドライブ文字またはパスのダイアログ](media/change-drive-letter3.png)