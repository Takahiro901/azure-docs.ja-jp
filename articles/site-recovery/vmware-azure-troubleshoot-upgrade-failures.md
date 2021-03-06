---
title: Microsoft Azure Site Recovery Provider のアップグレード エラーのトラブルシューティング | Microsoft Docs
description: 理解を深める
author: vDonGlover
manager: jarrettr
ms.service: site-recovery
ms.topic: troubleshooting
ms.date: 02/05/2019
ms.author: v-doglov
ms.openlocfilehash: 9423fc844e766129ad81a8a286cb5bbdc722e2ca
ms.sourcegitcommit: e69fc381852ce8615ee318b5f77ae7c6123a744c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/11/2019
ms.locfileid: "55989316"
---
# <a name="troubleshoot-microsoft-azure-site-recovery-provider-upgrade-failures"></a>Microsoft Azure Site Recovery プロバイダーのアップグレード エラーのトラブルシューティング

この記事では、Microsoft Azure Site Recovery Provider のアップグレード中にエラーを引き起こす可能性のある問題の解決について説明します。

## <a name="the-upgrade-fails-reporting-that-the-latest-site-recovery-provider-is-already-installed"></a>最新の Site Recovery Provider が既にインストールされていることを示すアップグレード エラー

Microsoft Azure Site Recovery Provider (DRA) をアップグレードするときに、統合セットアップのアップグレードが失敗して、エラー メッセージが表示されます。

さらに高いバージョンのソフトウェアが既にインストールされているため、アップグレードはサポートされません。

アップグレードするには、次の手順を使用します。

1. Microsoft Azure Site Recovery 統合セットアップをダウンロードします。
   1. 「[Azure Site Recovery のサービスの更新情報](service-updates-how-to.md##links-to-currently-supported-update-rollups)」記事の「現在サポートされている更新プログラム ロールアップへのリンク」セクションで、アップグレードするプロバイダーを選択します。
   2. ロールアップ ページで、**[Update information]\(情報の更新\)** セクションを探し、Microsoft Azure Site Recovery 統合セットアップの更新プログラムのロールアップをダウンロードします。

2. コマンド プロンプトを開き、統合セットアップ ファイルをダウンロードしたフォルダーに移動します。 「MicrosoftAzureSiteRecoveryUnifiedSetup.exe /q /x:&lt;抽出したファイルのフォルダー パス&gt;」というコマンドを使用して、ダウンロードからセットアップ ファイルを抽出します。
    
    コマンドの例:

    MicrosoftAzureSiteRecoveryUnifiedSetup.exe /q /x:C:\Temp\Extracted

3. コマンド プロンプトでファイルを抽出したフォルダーに移動し、次のインストール コマンドを実行します。
   
    CX_THIRDPARTY_SETUP.EXE /VERYSILENT /SUPPRESSMSGBOXES /NORESTART  UCX_SERVER_SETUP.EXE /VERYSILENT /SUPPRESSMSGBOXES /NORESTART /UPGRADE

1. 統合セットアップをダウンロードしたフォルダーに戻り、MicrosoftAzureSiteRecoveryUnifiedSetup.exe を実行してアップグレードを完了します。 

## <a name="upgrade-failure-due-to-the-thirdparty-folder-being-renamed"></a>thirdparty フォルダーの名前が変更されたためのアップグレード エラー

アップグレードが成功するには、thirdparty フォルダーの名前が変更されていてはなりません。

問題を解決するには、次のようにします。

2. レジストリ エディター (regedit.exe) を開始し、HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\InMage Systems\Installed Products\10 ブランチを開きます。
3. `Build_Version` キーの値を調べます。 最新バージョンに設定されている場合は、バージョン番号を下げます。 たとえば、最新バージョンが 9.22.\* で、`Build_Version` キーがその値に設定されている場合は、9.21.\* に下げます。
4. 最新の Microsoft Azure Site Recovery 統合セットアップをダウンロードします。
   1. 「[Azure Site Recovery のサービスの更新情報](service-updates-how-to.md##links-to-currently-supported-update-rollups)」記事の「現在サポートされている更新プログラム ロールアップへのリンク」セクションで、アップグレードするプロバイダーを選択します。
   2. ロールアップ ページで、**[Update information]\(情報の更新\)** セクションを探し、Microsoft Azure Site Recovery 統合セットアップの更新プログラムのロールアップをダウンロードします。
5. コマンド プロンプトを開き、統合セットアップ ファイルをダウンロードしたフォルダーに移動し、コマンド「MicrosoftAzureSiteRecoveryUnifiedSetup.exe /q /x:&lt;抽出したファイルのフォルダー パス&gt;」を使用してダウンロードからセットアップ ファイルを抽出します。

    コマンドの例:

    MicrosoftAzureSiteRecoveryUnifiedSetup.exe /q /x:C:\Temp\Extracted

4. コマンド プロンプトでファイルを抽出したフォルダーに移動し、次のインストール コマンドを実行します。
   
    CX_THIRDPARTY_SETUP.EXE /VERYSILENT /SUPPRESSMSGBOXES /NORESTART

5. タスク マネージャーを使用して、インストールの進行状況を監視します。 CX_THIRDPARTY_SETUP.EXE のプロセスがタスク マネージャーに表示されなくなったら、次のステップに進みます。
6. C:\thirdparty が存在し、そのフォルダーに RRD ライブラリが含まれていることを確認します。
1. 統合セットアップをダウンロードしたフォルダーに戻り、MicrosoftAzureSiteRecoveryUnifiedSetup.exe を実行してアップグレードを完了します。 