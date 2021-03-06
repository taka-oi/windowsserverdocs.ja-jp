---
title: 記憶域スペース ダイレクトのボリュームの拡張
description: Windows 管理センターと PowerShell を使用して記憶域スペースダイレクトのボリュームのサイズを変更する方法。
ms.prod: windows-server
ms.reviewer: cosmosdarwin
author: cosmosdarwin
ms.author: cosdar
manager: eldenc
ms.technology: storage-spaces
ms.date: 03/10/2020
ms.openlocfilehash: 4ce41da1da3dc90f698008902170d7cc1541619c
ms.sourcegitcommit: bb2eb0b12f2a32113899a59aa5644bc6e8cab3d2
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/11/2020
ms.locfileid: "79089348"
---
# <a name="extending-volumes-in-storage-spaces-direct"></a>記憶域スペース ダイレクトのボリュームの拡張
> 適用対象: Windows Server 2019、Windows Server 2016

このトピックでは、Windows 管理センターを使用して[記憶域スペースダイレクト](storage-spaces-direct-overview.md)クラスター上のボリュームのサイズを変更する手順について説明します。

> [!WARNING]
> **サポートされていません: 記憶域スペースダイレクトで使用される基になるストレージのサイズを変更します。** Azure に含まれる仮想化ストレージ環境で記憶域スペースダイレクトを実行している場合、仮想マシンが使用する記憶装置の特性のサイズ変更や変更はサポートされていないため、データにアクセスできなくなります。 代わりに、「[サーバーまたはドライブの追加](add-nodes.md)」セクションの手順に従って、ボリュームを拡張する前に容量を追加します。

ボリュームのサイズを変更する方法に関する簡単なビデオをご覧ください。

> [!VIDEO https://www.youtube-nocookie.com/embed/hqyBzipBoTI]

## <a name="extending-volumes-using-windows-admin-center"></a>Windows 管理センターを使用したボリュームの拡張

1. Windows 管理センターで、記憶域スペースダイレクトクラスターに接続し、 **[ツール]** ウィンドウで **[ボリューム]** を選択します。
2. ボリューム ページで、**インベントリ** タブを選択し、サイズを変更するボリュームを選択します。

    ボリュームの詳細ページで、ボリュームの記憶域容量が示されます。 [ボリュームの詳細] ページは、ダッシュボードから直接開くこともできます。 ダッシュボードの アラート ウィンドウで、アラート を選択します。これにより、ボリュームの記憶域容量が不足している場合に通知され、**ボリュームに移行する** を選択します。

4. ボリュームの詳細ページの上部で、 **[サイズ変更]** を選択します。
5. 新しいサイズを新しく入力し、 **[サイズ変更]** を選択します。

    [ボリュームの詳細] ページには、ボリュームの記憶域の容量が大きいことが示され、ダッシュボードのアラートがクリアされます。

## <a name="extending-volumes-using-powershell"></a>PowerShell を使用したボリュームの拡張

### <a name="capacity-in-the-storage-pool"></a>記憶域プールの容量

ボリュームのサイズを変更する前に、拡張された新しいフットプリントを収容するのに十分な容量が記憶域プールにあることを確認します。 たとえば、3 方向ミラーリング ボリュームを 1 TB から 2 TB に変更すると、そのフットプリントは 3 TB から 6 TB に増加します。 サイズ変更を正常に行うには、記憶域プールに使用か追うな容量が (6 - 3) = 3 TB 以上必要です。

### <a name="familiarity-with-volumes-in-storage-spaces"></a>記憶域スペース内のボリュームに関する知識

記憶域スペース ダイレクトでは、各ボリュームはスタックした複数のオブジェクト、つまりクラスター共有ボリューム (CSV) (ボリューム)、パーティション、ディスク (仮想ディスク)、1 つ以上の記憶域階層 (該当する場合) で構成されています。 ボリュームのサイズを変更するには、これらのオブジェクトのうち複数をサイズ変更する必要があります。

![SMAPI 内のボリューム](media/resize-volumes/volumes-in-smapi.png)

ボリュームについてよく知るため、PowerShell で対応する名詞を使って **Get-** を実行してみます。

例 :

```PowerShell
Get-VirtualDisk
```

スタック内のオブジェクト間の関連付けに従うには、パイプを使って **Get-** コマンドレットを次のコマンドレットに渡します。

たとえば、仮想ディスクからそのボリュームを取得する方法を次に示します。

```PowerShell
Get-VirtualDisk <FriendlyName> | Get-Disk | Get-Partition | Get-Volume 
```

### <a name="step-1--resize-the-virtual-disk"></a>手順 1 - 仮想ディスクのサイズ変更

仮想ディスクは、作成方法によって記憶域階層を使うことも使わないこともあります。

確認するには、次のコマンドレットを実行します。

```PowerShell
Get-VirtualDisk <FriendlyName> | Get-StorageTier 
```

コマンドレットにより何も返されない場合、仮想ディスクは記憶域階層を使っていません。

#### <a name="no-storage-tiers"></a>記憶域階層なし

仮想ディスクに記憶域階層がない場合、**Resize-VirtualDisk** コマンドレットを使って直接サイズ変更できます。

**-Size** パラメーターで新しいサイズを指定します。

```PowerShell
Get-VirtualDisk <FriendlyName> | Resize-VirtualDisk -Size <Size>
```

**VirtualDisk** をサイズ変更すると、**Disk** も自動的に追従し、サイズ変更されます。

![仮想ディスクのサイズ変更](media/resize-volumes/Resize-VirtualDisk.gif)

#### <a name="with-storage-tiers"></a>記憶域階層あり

仮想ディスクが記憶域階層を使っている場合、**Resize-StorageTier** コマンドレットを使って各階層を別個にサイズ変更できます。

仮想ディスクからの関連付けに従うことにより、記憶域階層の名前を取得します。

```PowerShell
Get-VirtualDisk <FriendlyName> | Get-StorageTier | Select FriendlyName
```

次に、階層ごとに **-Size** パラメーターで新しいサイズを指定します。

```PowerShell
Get-StorageTier <FriendlyName> | Resize-StorageTier -Size <Size>
```

> [!TIP]
> 階層の物理的なメディアの種類 (**MediaType = SSD** や **MediaType = HDD** など) が異なる場合、各階層の拡張された新しいフットプリントを収容する容量が、メディアの種類ごとに十分記憶域プール内にあることを確認する必要があります。

**StorageTier** をサイズ変更すると、**VirtualDisk** と **Disk** も自動的に追従し、サイズ変更されます。

![Resize-StorageTier](media/resize-volumes/Resize-StorageTier.gif)

### <a name="step-2--resize-the-partition"></a>手順 2 - パーティションのサイズ変更

次に、**Resize-Partition** コマンドレットを使ってパーティションのサイズを変更します。 仮想ディスクには 2 つのパーティションが必要です。1 つは、変更できない予約パーティション、もう 1 つは **PartitionNumber = 2** および **Type = Basic** が設定された、サイズ変更が必要なパーティションです。

**-Size** パラメーターで新しいサイズを指定します。 以下に示すように、サポートされる最大サイズを使うことをお勧めします。

```PowerShell
# Choose virtual disk
$VirtualDisk = Get-VirtualDisk <FriendlyName>

# Get its partition
$Partition = $VirtualDisk | Get-Disk | Get-Partition | Where PartitionNumber -Eq 2

# Resize to its maximum supported size 
$Partition | Resize-Partition -Size ($Partition | Get-PartitionSupportedSize).SizeMax
```

**Partition** をサイズ変更すると、**Volume** と **ClusterSharedVolume** も自動的に追従し、サイズ変更されます。

![パーティションのサイズ変更](media/resize-volumes/Resize-Partition.gif)

以上で終わりです。

> [!TIP]
> **Get-Volume** を実行して、ボリュームが新しいサイズになっていることを確認できます。

## <a name="see-also"></a>参照

- [Windows Server 2016 の記憶域スペースダイレクト](storage-spaces-direct-overview.md)
- [記憶域スペースダイレクトのボリュームの計画](plan-volumes.md)
- [記憶域スペースダイレクトでのボリュームの作成](create-volumes.md)
- [記憶域スペースダイレクトのボリュームの削除](delete-volumes.md)
