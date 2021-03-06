---
title: Project Acoustics Unreal のデザイン チュートリアル
titlesuffix: Azure Cognitive Services
description: このチュートリアルでは、Unreal と Wwiseにおける Project Acoustics のデザイン ワークフローについて説明します。
services: cognitive-services
author: kegodin
manager: nitinme
ms.service: cognitive-services
ms.subservice: acoustics
ms.topic: tutorial
ms.date: 03/20/2019
ms.author: kegodin
ms.openlocfilehash: 57bde67ac2259b3847f59f95eaefba9c6fddf13e
ms.sourcegitcommit: 90dcc3d427af1264d6ac2b9bde6cdad364ceefcc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/21/2019
ms.locfileid: "58316203"
---
# <a name="project-acoustics-unrealwwise-design-tutorial"></a>Project Acoustics Unreal/Wwise のデザイン チュートリアル
このチュートリアルでは、Unreal と Wwiseにおける Project Acoustics のデザイン セットアップとワークフローについて説明します。

ソフトウェアの前提条件:
* Project Acoustics Wwise および Unreal プラグインを使用する Unreal プロジェクト

Project Acoustics を使用して Unreal プロジェクトを取得するには、次のようにします。
* [Project Acoustics Unreal の統合](unreal-integration.md)の指示に従って、Project Acoustics を Unreal プロジェクトに追加する、
* または [Project Acoustics サンプル プロジェクト](unreal-quickstart.md)を使用します。

## <a name="setup-project-wide-wwise-properties"></a>プロジェクト全体の Wwise プロパティをセットアップする
Wwise には、Project Acoustics プラグインが Wwise オーディオ DSP を作動する方法に影響を与える、グローバルな障害曲線と閉鎖曲線があります。

### <a name="design-wwise-occlusion-curves"></a>Wwise 閉鎖曲線の設計
Project Acoustics がアクティブの場合は、Wwise で設定した閉鎖ボリューム、ローパス フィルター (LPF)、ハイパス フィルター (HPF) 曲線に応答します。 閉鎖値が 100 の場合、ボリューム カーブ タイプを -100 dB の値で線形に設定することをお勧めします。

この設定では、Project Acoustics シミュレーションが -18 dB の閉鎖を計算した場合、X=18 で下の曲線に入力され、対応する Y 値が適用される減衰量になります。 半分の閉鎖を実行するには、エンドポイントを -100 dB ではなく -50 dB に設定するか、閉鎖を強調するために -200 dB に設定します。 ゲームに最適な任意の曲線をカスタマイズしたり、微調整したりできます。
 
![Wwise 閉鎖曲線エディタのスクリーンショット](media/wwise-occlusion-curve.png)

### <a name="disable-wwise-obstruction-curves"></a>Wwise 障害曲線を無効にする
Wwise 閉塞曲線は単独でドライ レベルに影響しますが、Project Acoustics はデザイン コントロールとシミュレーションを使用してウェット/ドライの比率を適用します。 閉鎖ボリューム曲線を無効にすることをお勧めします。 ウェットを設計するには、後述の [ウェット調整] コントロールを使用します。
 
他の目的で障害 LPF/HPF 曲線を使用している場合は、X=0、Y=0 に設定してください (つまり、障害がない場合は LPF も HPF もありません)。

![Wwise 障害曲線エディタのスクリーンショット](media/wwise-obstruction-curve.png)

### <a name="design-project-acoustics-mixer-parameters"></a>Project Acoustics ミキサー パラメーターの設計
Project Acoustics Bus のミキサー プラグイン タブで、グローバル リバーブのプロパティをコントロールすることができます。 [Project Acoustics ミキサー (カスタム)] をダブルクリックしてミキサー プラグインの設定パネルを開きます。

ミキサー プラグインには [立体化の実行] オプションもあります。 Project Acoustic 内蔵の立体化を使用する場合は、[立体化の実行] チェックボックスをオンにして、[HRTF] または [パン] から選択します。 設定した [ドライ Aux] バスを無効にしてください。そうしないと、ダイレクト パスが 2 回聞こえてきます。 [ウェットの調整] と [リバーブ時間倍率] を使用してリバーブ ミックスをグローバルにコントロールします。 Unreal を再起動してから、再生を実行する前にサウンドバンクを再生成して、[立体化の実行] チェックボックスなどのミキサー プラグインの設定変更を選択する必要があります。

![Project Acoustics Wwise ミキサー プラグイン オプションのスクリーンショット](media/mixer-plugin-global-settings.png)

## <a name="set-project-acoustics-design-controls-in-the-wwise-actor-mixer-hierarchy"></a>Wwise アクターミキサー階層で Project Acoustics デザイン コントロールを設定する
個々のアクターミキサーのパラメーターをコントロールするには、[アクターミキサー] をダブルクリックしてから、その [ミキサー プラグイン] タブをクリックします。ここでは、サウンド レベルごとに任意のパラメーターを変更することができます。 これらの値は、Unreal 側から設定された値と結合されます (以下で説明します)。 たとえば、Project Acoustics Unreal プラグインでオブジェクトの[Outdoorness の調整] を 0.5 に設定し、Wwise で -0.25 に設定した場合、そのサウンドに適用される [Outdoorness の調整] は 0.25 になります。

![Wwise アクターミキサー階層におけるサウンドごとのミキサー設定のスクリーンショット](media/per-sound-mixer-settings.png)

### <a name="ensure-the-aux-bus-has-dry-send-and-output-bus-has-wet-send"></a>Aux バスがドライ送信、出力バスがウェット送信であることを確認する
必要なアクターミキサー設定では、Wwise の通常のドライとウェット ルーティングが交換されることに注意してください。 これにより、アクターミキサーの出力バス (Project Acoustics Bus に設定される) でリバーブ シグナルが、ユーザー定義の Aux バスに沿ってドライ シグナルが生成されます。 このルーティングは、Project Acoustics Wwise プラグインが使用する Wwise ミキサー プラグイン API の機能のために必要です。

![Project Acoustics の音声設計ガイドラインを示す Wwise エディターのスクリーンショット](media/voice-design-guidelines.png)
 
### <a name="set-up-distance-attenuation-curves"></a>距離減衰曲線の設定
Project Acoustics を使用するアクターミキサーで使用されている減衰曲線で、ユーザー定義の AUX 送信が [出力バス ボリューム] に設定されていることを確認します。 既定で、Wwise は新しく作成された減衰曲線に対してこれを実行します。 既存のプロジェクトを移行する場合は、曲線設定を確認します。 

既定では、Project Acoustics シミュレーションはプレーヤーの場所から半径 45 メートルに設定されています。 通常、減衰曲線をその距離付近で -200 dB に設定することをお勧めします。 この距離は、厳しい制約ではありません。 武器のような音の場合、より大きな半径を設定することができます。 そのような場合、プレーヤーの位置から 45 m以内のジオメトリだけが含まれるという点で注意が必要です。 プレーヤーが部屋の中にいて、音源が部屋の外にあり、100m 離れている場合は適切に遮断されます。 ソースが部屋の中にあり、プレーヤーが外にいて 100 m 離れている場合は適切に遮断されません。

![Wwise 減衰曲線のスクリーンショット](media/atten-curve.png)

## <a name="set-up-scene-wide-project-acoustics-properties"></a>シーン全体の Project Acoustics プロパティを設定する

Acoustics Space アクターは、システムの動作を変更し、デバッグに役立つ多くのコントロールを提供します。

![Unreal Acoustics Space コントロールのスクリーンショット](media/acoustics-space-controls.png)

* **音響データ:** このフィールドには、[Content/Acoustics] ディレクトリからベイクされた音響アセットを割り当てる必要があります。 Project Acoustics プラグインはプロジェクトのパッケージ ディレクトリに [Content/Acoustics] ディレクトリを自動的に追加します。
* **タイル サイズ:** 音響データを RAM にロードするリスナー周辺の領域の範囲。 プレーヤのすぐ周りにあるリスナー プローブがロードされている限り、結果はすべてのプローブの音響データをロードした場合と同じになります。 タイルが大きくなればなるほど RAM の使用率は増えるが、ディスク I/O は減少する
* **自動ストリーム:** 有効にすると、リスナーがロードされた領域の端に達すると自動的に新しいタイルがロードされます。 無効になっている場合は、コードまたはブループリントを使用して新しいタイルを手動でロードする必要があります。
* **キャッシュ スケール:** 音響クエリに使用されるキャッシュのサイズを制御します。 キャッシュを小さくすると RAM の使用率は少なくなりますが、クエリごとの CPU 使用率が増加する可能性があります。
* **Acoustics 対応:** Acoustics シミュレーションの迅速な A/B 切り替えを可能にするデバッグ コントロール。 配布構成では、このコントロールは無視されます。 このコントロールは、特定のオーディオ バグが音響計算または Wwise プロジェクト内の他の問題のどちらに由来するのかを判断するのに役立ちます。
* **距離の更新:** 距離のクエリに事前にベイクされた音響情報を使用する場合に、このオプションを使用します。 これらのクエリはレイ キャストと同じですが、事前に計算されているため、CPU の使用量ははるかに少なくて済みます。 使用例として、リスナーに最も近いサーフェスからの不連続な反射が挙げられます。 これを十分に活用するには、コードまたはブループリントを使用して距離をクエリする必要があります。
* **統計の描画:** UE の `stat Acoustics` は CPU 情報を提供しますが、この状態表示では、現在ロードされているマップ、RAM 使用量、およびその他の状態情報が画面の左上に表示されます。
* **ボクセルの描画:** ランタイム補間中に使用されたボクセル グリッドを示す、リスナーに近いボクセルをオーバーレイします。 エミッタがランタイム ボクセル内にあると、音響クエリは失敗します。
* **プローブの描画:** このシーンのすべてのプローブを表示します。 負荷状態に応じて異なる色があります。
* **距離の描画:**[距離の更新] が有効になっている場合、リスナーの周囲の量子化された方向にいるリスナーに最も近いサーフェス上に、ボックスが表示されます。

## <a name="actor-specific-acoustics-design-controls"></a>アクター固有の音響設計コントロール
これらの設計コントロールは、Unreal の個々のオーディオ コンポーネントを対象としています。

![Unreal オーディオ コンポーネント コントロールのスクリーンショット](media/audio-component-controls.png)

* **オクルージョン乗数:** オクルージョン効果をコントロールします。 値 1 > の場合、オクルージョンが増幅されます。 値 < 1 の場合、オクルージョンが最小限に抑えられます。
* **ウェット調整:** 追加のリバーブ dB
* **減退時間の乗数:** Acoustics シミュレーションの出力に基づいて RT60 を乗法的に制御します
* **Outdoorness 調整:** 屋外での残響の程度を制御します。 値が 0 に近いほど室内で、1 に近いほど屋外です。 この調整は加法的であり、-1 に設定すると室内が適用され、+1 に設定すると屋外が適用されます。
* **伝送 Db:** 見通し内をベースにした距離減衰と組み合わせたこのラウドネスを使用して、追加のスルーザウォール サウンドをレンダリングします。
* **ウェット比率距離ワープ:** ダイレクト パスに影響を与えずに、ソースの残響特性を近づけたり遠ざけたりするように調整します。
* **音響パラメーターの表示:** ゲーム内のコンポーネントの上部にデバッグ情報を直接表示します。 (非配布構成の場合のみ)

## <a name="blueprint-functionality"></a>ブループリント機能
Acoustics Space アクターはブループリントからアクセス可能であり、マップのロードやレベル スクリプトによる設定変更などの機能を提供します。 ここで、2 つの例を紹介します。

### <a name="add-finer-grained-control-over-streaming-load"></a>ストリーミング負荷をきめ細かく制御する
プレーヤーの位置に基づいて自動的にストリーミングするのではなく、自分でストリーミングする音響データを管理するには、[タイルの強制読み込み] ブループリント機能を使用できます。

![Unreal のブループリント ストリーミング オプションのスクリーンショット](media/blueprint-streaming.png)

* **ターゲット:** AcousticsSpace アクター
* **中央の位置:** データを読み込む必要がある領域の中心
* **タイル外のプローブのアンロード:** チェックすると、新しい領域にないすべてのプローブが RAM からアンロードされます。 チェックを解除すると、既存のプローブもメモリにロードされたまま、新しい領域がメモリにロードされます
* **完了時にブロック:** タイルに同期操作をロードさせます

タイル サイズは [タイルの強制読み込み] を呼び出す前に設定されている必要があります。 たとえば、ACE ファイルをロードし、タイル サイズを設定し、領域でストリーミングするには、次のようにします。

![Unreal のストリーミング設定オプションのスクリーンショット](media/streaming-setup.png)

### <a name="optionally-query-for-surface-proximity"></a>オプションでサーフェスの近接をクエリする
サーフェスがリスナーの周囲の特定の方向にどれだけ近づいているかを知りたい場合は、[距離のクエリ] 機能を使用できます。 この機能は、指向性の遅延反射を実施する場合や、サーフェスの近接によって実施されるその他のゲーム ロジックに役立ちます。 クエリは音響ルックアップ テーブルから取得されるため、クエリはレイキャストよりも低いコストになります。

![サンプル ブループリント距離クエリのスクリーンショット](media/distance-query.png)

* **ターゲット:** AcousticsSpace アクター
* **検索方向:** リスナーを中心とした、クエリする方向
* **距離:** クエリが成功した場合の、最も近いサーフェスまでの距離
* **戻り値:** ブール値 - クエリが成功した場合は true、それ以外の場合は false

## <a name="next-steps"></a>次の手順
* [設計プロセス](design-process.md)の背後にある概念を確認する
* 独自のシーンをベイクするための [Azure アカウントを作成する](create-azure-account.md)


