---
title: Azure IoT Hub デバイス ストリーム C クイック スタート (プレビュー) | Microsoft Docs
description: このクイック スタートでは、デバイス ストリームを介して IoT デバイスと通信する C サービス側アプリケーションを実行します。
author: rezasherafat
manager: briz
ms.service: iot-hub
services: iot-hub
ms.devlang: c
ms.topic: quickstart
ms.custom: mvc
ms.date: 03/14/2019
ms.author: rezas
ms.openlocfilehash: 6a0fd87c787108935430ca43310a662418833c96
ms.sourcegitcommit: 0dd053b447e171bc99f3bad89a75ca12cd748e9c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/26/2019
ms.locfileid: "58480755"
---
# <a name="quickstart-communicate-to-a-device-application-in-c-via-iot-hub-device-streams-preview"></a>クイック スタート:IoT Hub デバイス ストリームを介して C でデバイス アプリケーションと通信する (プレビュー)

[!INCLUDE [iot-hub-quickstarts-3-selector](../../includes/iot-hub-quickstarts-3-selector.md)]

Microsoft Azure IoT Hub は現在、[プレビュー機能](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)としてデバイス ストリームをサポートしています。

[IoT Hub デバイス ストリーム](./iot-hub-device-streams-overview.md)を使用すると、サービス アプリケーションとデバイス アプリケーションが、安全でファイアウォールに対応した方法で通信できます。 パブリック プレビュー中、C SDK ではデバイス側のデバイス ストリームのみがサポートされます。 そのため、このクイック スタートでは、デバイス側アプリケーションを実行する手順についてのみ説明しています。 対応するサービス側アプリケーションも実行する必要があり、それについては [C# クイック スタート](./quickstart-device-streams-echo-csharp.md)または [Node.js クイック スタート](./quickstart-device-streams-echo-nodejs.md)に記載されています。

このクイック スタートのデバイス側 C アプリケーションには、以下の機能があります。

* IoT デバイスへのデバイス ストリームを確立します。

* サービス側から送信されたデータを受信し、それをエコーバックします。

コードは、デバイス ストリームの開始プロセスと、それを使用してデータを送受信する方法を示しています。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Azure サブスクリプションがない場合は、開始する前に[無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)を作成してください。

## <a name="prerequisites"></a>前提条件

* デバイス ストリームのプレビューは現在、次のリージョンで作成された IoT Hub に対してのみサポートされています。

  * **米国中部**
  * **米国中部 EUAP**

* ["C++ によるデスクトップ開発"](https://www.visualstudio.com/vs/support/selecting-workloads-visual-studio-2017/) ワークロードを有効にした [Visual Studio 2017](https://www.visualstudio.com/vs/) をインストールします。
* 最新バージョンの [Git](https://git-scm.com/download/) をインストールします。

## <a name="prepare-the-development-environment"></a>開発環境の準備

このクイック スタートでは、[C 対応の Azure IoT device SDK](iot-hub-device-sdk-c-intro.md) を使用します。[Azure IoT C SDK](https://github.com/Azure/azure-iot-sdk-c) を GitHub から複製してビルドするために使用される開発環境を準備します。 GitHub 上の SDK には、このクイック スタートで使用されるサンプル コードが含まれます。 

1. [CMake ビルド システム](https://cmake.org/download/)をダウンロードします。 ダウンロードしたバイナリを、ダウンロードしたバージョンに対応する暗号化ハッシュ値を使用して検証します。 暗号化ハッシュ値も、既に示した CMake のダウンロード リンクの場所にあります。

    次の例では、Windows PowerShell を使用して、x64 MSI 配布のバージョン 3.13.4 の暗号化ハッシュを検証しています。

    ```powershell
    PS C:\Downloads> $hash = get-filehash .\cmake-3.13.4-win64-x64.msi
    PS C:\Downloads> $hash.Hash -eq "64AC7DD5411B48C2717E15738B83EA0D4347CD51B940487DFF7F99A870656C09"
    True
    ```

    この記事の執筆時点では、CMake サイトにバージョン 3.13.4 用に次のハッシュ値が一覧表示されていました。

    ```
    563a39e0a7c7368f81bfa1c3aff8b590a0617cdfe51177ddc808f66cc0866c76  cmake-3.13.4-Linux-x86_64.tar.gz
    7c37235ece6ce85aab2ce169106e0e729504ad64707d56e4dbfc982cb4263847  cmake-3.13.4-win32-x86.msi
    64ac7dd5411b48c2717e15738b83ea0d4347cd51b940487dff7f99a870656c09  cmake-3.13.4-win64-x64.msi
    ```

    `CMake` のインストールを開始する**前に**、Visual Studio の前提条件 (Visual Studio と "C++ によるデスクトップ開発" ワークロード) が マシンにインストールされていることが重要です。 前提条件を満たし、ダウンロードを検証したら、CMake ビルド システムをインストールします。

2. コマンド プロンプトまたは Git Bash シェルを開きます。 次のコマンドを実行して、[Azure IoT C SDK](https://github.com/Azure/azure-iot-sdk-c) の GitHub リポジトリを複製します。
    
    ```
    git clone https://github.com/Azure/azure-iot-sdk-c.git --recursive -b public-preview
    ```

    このリポジトリのサイズは現在約 220 MB です。

3. git リポジトリのルート ディレクトリに `cmake` サブディレクトリを作成し、そのフォルダーに移動します。 

    ```
    cd azure-iot-sdk-c
    mkdir cmake
    cd cmake
    ```

4. `cmake` ディレクトリから次のコマンドを実行して、開発クライアント プラットフォームに固有の SDK のバージョンをビルドします。

   * Linux の場合:

      ```bash
      cmake ..
      make -j
      ```

   * Windows では、Visual Studio 2015 または 2017 用の開発者コマンド プロンプトで、次のコマンドを実行します。 シミュレートされたデバイスの Visual Studio ソリューションが `cmake` ディレクトリに生成されます。

      ```cmd
      rem For VS2015
      cmake .. -G "Visual Studio 14 2015"

      rem Or for VS2017
      cmake .. -G "Visual Studio 15 2017"

      rem Then build the project
      cmake --build . -- /m /p:Configuration=Release
      ```

## <a name="create-an-iot-hub"></a>IoT Hub の作成

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub-device-streams.md)]

## <a name="register-a-device"></a>デバイスの登録

デバイスを IoT ハブに接続するには、あらかじめ IoT ハブに登録しておく必要があります。 このセクションでは、[IoT 拡張機能](https://docs.microsoft.com/cli/azure/ext/azure-cli-iot-ext/iot?view=azure-cli-latest)と共に Azure Cloud Shell を使用して、シミュレートされたデバイスを登録します。

1. Azure Cloud Shell で次のコマンドを実行して IoT Hub CLI 拡張機能を追加し、デバイス ID を作成します。 

   **YourIoTHubName**: このプレースホルダーは、実際の IoT ハブに対して選んだ名前に置き換えてください。

   **MyDevice**: これは、登録済みデバイスに付けられた名前です。 示されているように、MyDevice を使用します。 デバイスに別の名前を選択した場合は、この記事全体でその名前を使用する必要があります。また、サンプル アプリケーションを実行する前に、アプリケーション内のデバイス名を更新してください。

    ```azurecli-interactive
    az extension add --name azure-cli-iot-ext
    az iot hub device-identity create --hub-name YourIoTHubName --device-id MyDevice
    ```

2. Azure Cloud Shell で次のコマンドを実行して、登録したデバイスの "_デバイス接続文字列_" を取得します。

   **YourIoTHubName**: このプレースホルダーは、実際の IoT ハブに対して選んだ名前に置き換えてください。

    ```azurecli-interactive
    az iot hub device-identity show-connection-string --hub-name YourIoTHubName --device-id MyDevice --output table
    ```

    次の例のような、デバイスの接続文字列をメモしておきます。

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyDevice;SharedAccessKey={YourSharedAccessKey}`

    この値は、このクイック スタートの後の方で使います。

## <a name="communicate-between-device-and-service-via-device-streams"></a>デバイス ストリームを介したデバイスとサービスの間の通信

### <a name="run-the-device-side-application"></a>デバイス側アプリケーションの実行

デバイス側アプリケーションを実行するには、次の手順を実行する必要があります。

1. ソース ファイル `iothub_client/samples/iothub_client_c2d_streaming_sample/iothub_client_c2d_streaming_sample.c` を編集してデバイスの資格情報を指定し、デバイスの接続文字列を指定します。

   ```C
   /* Paste in your iothub connection string  */
   static const char* connectionString = "[device connection string]";
   ```

2. 次のようにコードをコンパイルします。

   ```bash
   # In Linux
   # Go to the sample's folder cmake/iothub_client/samples/iothub_client_c2d_streaming_sample
   make -j
   ```

   ```cmd
   rem In Windows
   rem Go to the cmake folder at the root of repo
   cmake --build . -- /m /p:Configuration=Release
   ```

3. コンパイルされたプログラムを実行します。

   ```bash
   # In Linux
   # Go to the sample's folder cmake/iothub_client/samples/iothub_client_c2d_streaming_sample
   ./iothub_client_c2d_streaming_sample
   ```

   ```cmd
   rem In Windows
   rem Go to the sample's release folder cmake\iothub_client\samples\iothub_client_c2d_streaming_sample\Release
   iothub_client_c2d_streaming_sample.exe
   ```

### <a name="run-the-service-side-application"></a>サービス側アプリケーションの実行

前述のように、IoT Hub C SDK ではデバイス側のデバイス ストリームのみがサポートされます。 サービス側アプリケーションを構築して実行するには、[C# クイック スタート](./quickstart-device-streams-echo-csharp.md)または [Node.js クイック スタート](./quickstart-device-streams-echo-nodejs.md)に記載されている手順に従ってください。

## <a name="clean-up-resources"></a>リソースのクリーンアップ

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources-device-streams.md)]

## <a name="next-steps"></a>次の手順

このクイック スタートでは、IoT ハブの設定、デバイスの登録、デバイス上の C アプリケーションとサービス側の別のアプリケーション間のデバイス ストリームの確立、そのストリームを使用したアプリケーション間のデータのやり取りを行いました。

以下のリンクを使用して、デバイス ストリームについてさらに詳しく学習します。

> [!div class="nextstepaction"]
> [デバイス ストリームの概要](./iot-hub-device-streams-overview.md)