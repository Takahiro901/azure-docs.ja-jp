---
title: テンプレートを使用してワークスペースを作成する
titleSuffix: Azure Machine Learning service
description: Azure Resource Manager テンプレートを使用して新しい Azure Machine Learning service ワークスペースを作成する方法について説明します。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: larryfr
author: Blackmist
ms.date: 02/11/2019
ms.openlocfilehash: 2b5ee1b5079f22985b03aaafac30c11a10a3c8f0
ms.sourcegitcommit: 5839af386c5a2ad46aaaeb90a13065ef94e61e74
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/19/2019
ms.locfileid: "57850079"
---
# <a name="create-an-azure-machine-learning-service-workspace-by-using-a-template"></a>テンプレートを使用して Azure Machine Learning service ワークスペースを作成する

この記事では、Azure Resource Manager テンプレートを使用して Azure Machine Learning service ワークスペースを作成するさまざまな方法について説明します。 

詳細については、「 [Azure リソース マネージャーのテンプレートを使用したアプリケーションのデプロイ](../../azure-resource-manager/resource-group-template-deploy.md)」を参照してください。

## <a name="prerequisites"></a>前提条件

* **Azure サブスクリプション**。 お持ちでない場合は、[無料版または有料版の Azure Machine Learning service](https://aka.ms/AMLFree) をお試しください。

* CLI からテンプレートを使用するには、[Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview?view=azps-1.2.0) または [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) が必要です。

## <a name="resource-manager-template"></a>Resource Manager テンプレート

Resource Manager テンプレートを使用すると、1 つの調整された操作でリソースを簡単に作成できます。 テンプレートは、デプロイに必要なリソースを定義する JSON ドキュメントです。 デプロイ パラメーターを指定することもできます。 パラメーターは、テンプレートの使用時に入力値を指定するために使用します。

次のテンプレートを使用すると、Azure Machine Learning service ワークスペースおよび関連する Azure リソースを作成できます。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceName": {
            "type": "string",
            "metadata": {
                "description": "The name of the Azure Machine Learning service workspace."
            }
        },
        "location": {
            "type": "string",
            "allowedValues": [
                "eastus",
                "eastus2",
                "southcentralus",
                "southeastasia",
                "westcentralus",
                "westeurope",
                "westus2"
            ],
            "metadata": {
                "description": "The location where the workspace will be created."
            }
        }
    },
    "variables": {
        "storageAccount": {
            "name": "[concat('sa',uniqueString(resourceGroup().id))]",
            "type": "Standard_LRS"
        },
        "keyVault": {
            "name": "[concat('kv',uniqueString(resourceGroup().id))]",
            "tenantId": "[subscription().tenantId]"
        },
        "applicationInsightsName": "[concat('ai',uniqueString(resourceGroup().id))]",
        "containerRegistryName": "[concat('cr',uniqueString(resourceGroup().id))]"
    },
    "resources": [
        {
            "name": "[variables('storageAccount').name]",
            "type": "Microsoft.Storage/storageAccounts",
            "location": "[parameters('location')]",
            "apiVersion": "2018-07-01",
            "sku": {
                "name": "[variables('storageAccount').type]"
            },
            "kind": "StorageV2",
            "properties": {
                "encryption": {
                    "services": {
                        "blob": {
                            "enabled": true
                        },
                        "file": {
                            "enabled": true
                        }
                    },
                    "keySource": "Microsoft.Storage"
                },
                "supportHttpsTrafficOnly": true
            }
        },
        {
            "name": "[variables('keyVault').name]",
            "type": "Microsoft.KeyVault/vaults",
            "apiVersion": "2018-02-14",
            "location": "[parameters('location')]",
            "properties": {
                "tenantId": "[variables('keyVault').tenantId]",
                "sku": {
                    "name": "standard",
                    "family": "A"
                },
                "accessPolicies": []
            }
        },
        {
            "name": "[variables('applicationInsightsName')]",
            "type": "Microsoft.Insights/components",
            "apiVersion": "2015-05-01",
            "location": "[if(or(equals(parameters('location'),'eastus2'),equals(parameters('location'),'westcentralus')),'southcentralus',parameters('location'))]",
            "kind": "web",
            "properties": {
                "Application_Type": "web"
            }
        },
        {
            "name": "[variables('containerRegistryName')]",
            "type": "Microsoft.ContainerRegistry/registries",
            "apiVersion": "2017-10-01",
            "location": "[parameters('location')]",
            "sku": {
                "name": "Standard"
            },
            "properties": {
                "adminUserEnabled": true
            }
        },
        {
            "name": "[parameters('workspaceName')]",
            "type": "Microsoft.MachineLearningServices/workspaces",
            "apiVersion": "2018-11-19",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[variables('storageAccount').name]",
                "[variables('keyVault').name]",
                "[variables('applicationInsightsName')]",
                "[variables('containerRegistryName')]"
            ],
            "identity": {
                "type": "systemAssigned"
            },
            "properties": {
                "friendlyName": "[parameters('workspaceName')]",
                "keyVault": "[resourceId('Microsoft.KeyVault/vaults',variables('keyVault').name)]",
                "applicationInsights": "[resourceId('Microsoft.Insights/components',variables('applicationInsightsName'))]",
                "containerRegistry": "[resourceId('Microsoft.ContainerRegistry/registries',variables('containerRegistryName'))]",
                "storageAccount": "[resourceId('Microsoft.Storage/storageAccounts/',variables('storageAccount').name)]"
            }
        }
    ]
}
```

このテンプレートでは、次の Azure サービスが作成されます。

* Azure リソース グループ
* Azure Storage アカウント
* Azure Key Vault
* Azure Application Insights
* Azure Container Registry
* Azure Machine Learning ワークスペース

リソース グループは、サービスを保持するコンテナーです。 Azure Machine Learning ワークスペースにはさまざまなサービスが必要です。

サンプルのテンプレートには次に示す 2 つのパラメーターがあります。

* リソース グループとサービスを作成する**場所**。

    このテンプレートでは、大部分のリソース用に選択する場所が使用されます。 例外は Application Insights サービスです。このサービスは、他のサービスが存在するすべての場所で使用できません。 使用できない場所を選択すると、米国中南部の場所にサービスが作成されます。

* **ワークスペース名**。これは Azure Machine Learning ワークスペースのフレンドリ名です。

    その他のサービスの名前はランダムに生成されます。

テンプレートの詳細については、次の記事を参照してください。

* [Azure リソース マネージャーのテンプレートの作成](../../azure-resource-manager/resource-group-authoring-templates.md)
* [Azure Resource Manager テンプレートを使用したアプリケーションのデプロイ](../../azure-resource-manager/resource-group-template-deploy.md)
* [Microsoft.MachineLearningServices resource types (Microsoft.MachineLearningServices リソースの種類)](https://docs.microsoft.com/azure/templates/microsoft.machinelearningservices/allversions)

## <a name="use-the-azure-portal"></a>Azure ポータルの使用

1. 「[カスタム テンプレートからリソースをデプロイする](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy-portal#deploy-resources-from-custom-template)」の手順に従います。 __[テンプレートの編集]__ 画面に到達したら、このドキュメントからテンプレートを貼り付けます。
1. __[保存]__ を選択してテンプレートを使用します。 次の情報を指定して、表示される使用条件に同意します。

   * サブスクリプション:これらのリソースに使用する Azure サブスクリプションを選択します。
   * リソース グループ: サービスが含まれるリソース グループを選択または作成します。
   * ワークスペース名: 作成される Azure Machine Learning ワークスペースに使用する名前。 ワークスペース名は 3 から 33 文字で指定する必要があります。 使用できるのは英数字と "-" のみです。
   * 場所:リソースを作成する場所を選択します。

     ![Azure portal 内のテンプレート パラメーター](media/how-to-create-workspace-template/template-parameters.png)

詳細については、「[カスタム テンプレートからリソースをデプロイする](../../azure-resource-manager/resource-group-template-deploy-portal.md#deploy-resources-from-custom-template)」を参照してください。

## <a name="use-azure-powershell"></a>Azure PowerShell の使用

この例では、現在のディレクトリ内の `azuredeploy.json` という名前のファイルにテンプレートを保存したと仮定します。

```powershell
New-AzResourceGroup -Name examplegroup -Location "East US"
new-azresourcegroupdeployment -name exampledeployment `
  -resourcegroupname examplegroup -location "East US" `
  -templatefile .\azuredeploy.json -workspaceName "exampleworkspace"
```

詳細については、「[Resource Manager テンプレートと Azure PowerShell を使用したリソースのデプロイ](../../azure-resource-manager/resource-group-template-deploy.md)」と「[SAS トークンと Azure PowerShell を使用してプライベートの Resource Manager テンプレートをデプロイする](../../azure-resource-manager/resource-manager-powershell-sas-token.md)」を参照してください。

## <a name="use-azure-cli"></a>Azure CLI の使用

この例では、現在のディレクトリ内の `azuredeploy.json` という名前のファイルにテンプレートを保存したと仮定します。

```azurecli-interactive
az group create --name examplegroup --location "East US"
az group deployment create \
  --name exampledeployment \
  --resource-group examplegroup \
  --template-file azuredeploy.json \
  --parameters workspaceName=exampleworkspace
```

詳細については、「[Resource Manager テンプレートと Azure CLI を使用したリソースのデプロイ](../../azure-resource-manager/resource-group-template-deploy-cli.md)」と「[SAS トークンと Azure CLI を使用してプライベートの Resource Manager テンプレートをデプロイする](../../azure-resource-manager/resource-manager-cli-sas-token.md)」を参照してください。

## <a name="next-steps"></a>次の手順

* [Resource Manager テンプレートと Resource Manager REST API を使用したリソースのデプロイ](../../azure-resource-manager/resource-group-template-deploy-rest.md)。
* [Visual Studio での Azure リソース グループの作成とデプロイ](../../azure-resource-manager/vs-azure-tools-resource-groups-deployment-projects-create-deploy.md)。
