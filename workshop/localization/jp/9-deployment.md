# Azure Container Apps への .NET Aspire アプリのデプロイ

.NET Aspire は、コンテナ化環境での実行を目的としたアプリケーション向けに最適化されています。[Azure Container Apps](https://learn.microsoft.com/azure/container-apps/overview) は、サーバーレス プラットフォームでマイクロサービスやコンテナ化されたアプリケーションを実行できる完全管理型環境です。この記事では、新しい .NET Aspire ソリューションを作成し、Visual Studio と Azure Developer CLI (`azd`) を使用して Microsoft Azure Container Apps にデプロイする方法を説明します。

この例では、前のセクションで作成した MyWeatherHub アプリをデプロイすることを前提としています。自分で構築したコードを使用するか、**complete** ディレクトリのコードを使用できます。ただし、一般的な手順はどの .NET Aspire アプリでも同じです。

## Visual Studio を使用してアプリをデプロイ

1. ソリューション エクスプローラーで **AppHost** プロジェクトを右クリックし、**発行** を選択して **発行** ダイアログを開きます。

    > .NET Aspire の発行には、現在のバージョンの `azd` CLI が必要です。これは .NET Aspire ワークロードと一緒にインストールされるはずですが、CLI がインストールされていない、または最新でないという通知を受け取った場合は、このチュートリアルの次の部分の手順に従ってインストールできます。

1. 発行ターゲットとして **Azure Container Apps for .NET Aspire** を選択します。

    ![発行ダイアログ ワークフローのスクリーンショット。](media/vs-deploy.png)

1. **AzDev Environment** ステップで、希望する **サブスクリプション** と **場所** の値を選択し、_aspire-weather_ などの **環境名** を入力します。環境名は、Azure Container Apps 環境リソースの命名を決定します。
1. **完了** を選択して環境を作成し、**閉じる** を選択してダイアログ ワークフローを終了し、デプロイ環境の概要を表示します。
1. **発行** を選択して、Azure でリソースをプロビジョニングしてデプロイします。

    > このプロセスの完了には数分かかる場合があります。Visual Studio は出力ログでデプロイの進行状況の状態更新を提供し、これらの更新を観察することで発行の動作について多くを学ぶことができます！プロセスには、リソース グループ、Azure Container Registry、Log Analytics ワークスペース、Azure Container Apps 環境の作成が含まれることがわかります。その後、アプリが Azure Container Apps 環境にデプロイされます。

1. 発行が完了すると、Visual Studio は環境画面の下部にリソース URL を表示します。これらのリンクを使用して、デプロイされたさまざまなリソースを表示します。**webfrontend** URL を選択して、デプロイされたアプリへのブラウザーを開きます。

    ![完了した発行プロセスとデプロイされたリソースのスクリーンショット。](media/vs-publish-complete.png)

## Azure Developer CLI (azd) を使用してアプリをデプロイ

### Azure Developer CLI のインストール

`azd` をインストールするプロセスはオペレーティング システムによって異なりますが、`winget`、`brew`、`apt`、または `curl` を介して直接広く利用できます。`azd` をインストールするには、[Azure Developer CLI のインストール](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd) を参照してください。

### .NET Aspire 9.4 の新機能：aspire deploy コマンド

.NET Aspire 9.4 では、発行機能を拡張してターゲット環境に積極的にデプロイする `aspire deploy` コマンド（プレビュー/機能フラグ）が導入されました。このコマンドは、カスタムのプリ/ポストデプロイロジックを使用した拡張デプロイ ワークフローを提供します。

この機能を有効にするには：

```bash
aspire config set features.deployCommandEnabled true
```

その後、次を使用できます：

```bash
aspire deploy
```

このコマンドは、進行状況レポートの向上、より良いエラー メッセージ、複雑なデプロイ シナリオ用のカスタム デプロイ フックのサポートを提供します。

### テンプレートの初期化

> 前提条件：
>
> - サインインしていることを確認してください：`azd login` を実行し、正しい Azure サブスクリプションを選択します。
> - AppHost を含むフォルダーから次のコマンドを実行します（このリポジトリの場合、完成したサンプルをデプロイする場合は通常 `complete` フォルダー）。

1. 新しいターミナル ウィンドウを開き、.NET Aspire プロジェクトのルートに `cd` します。
1. `azd init` コマンドを実行してプロジェクトを `azd` で初期化します。これにより、ローカル ディレクトリ構造が検査され、アプリの種類が決定されます。

    ```console
    azd init
    ```

    `azd init` コマンドの詳細については、[azd init](https://learn.microsoft.com/azure/developer/azure-developer-cli/reference#azd-init) を参照してください。
1. アプリを初めて初期化する場合、`azd` は環境名の入力を求めます：

    ```console
    Azure で実行するアプリの初期化 (azd init)
    
    ? 新しい環境名を入力してください： [? でヘルプ]
    ```

    希望する環境名を入力して続行します。`azd` での環境管理の詳細については、[azd env](https://learn.microsoft.com/azure/developer/azure-developer-cli/reference#azd-env) を参照してください。
1. `azd` が 2 つのアプリ初期化オプションを提示したら、**現在のディレクトリのコードを使用** を選択します。

    ```console
    ? アプリをどのように初期化しますか？  [矢印で移動、タイプでフィルター]
    > 現在のディレクトリのコードを使用
      テンプレートを選択
    ```

1. ディレクトリをスキャンした後、`azd` は正しい .NET Aspire _AppHost_ プロジェクトが見つかったことの確認を求めます。**確認してアプリの初期化を続行** オプションを選択します。

    ```console
    検出されたサービス：
    
      .NET (Aspire)
      検出場所： D:\source\repos\letslearn-dotnet-aspire\complete\AppHost\AppHost.csproj
    
    azd は Azure Container Apps を使用して Azure でアプリをホストするために必要なファイルを生成します。
    
    ? オプションを選択してください  [矢印で移動、タイプでフィルター]
    > 確認してアプリの初期化を続行
      キャンセルして終了
    ```

1. `azd` は .NET Aspire ソリューション内の各プロジェクトを表示し、すべてのインターネット トラフィックに公開的に開かれた HTTP イングレスでデプロイするものを特定するよう求めます。API (`api`) を Azure Container Apps 環境のプライベートにして、公開的に利用できないようにしたいため、`myweatherhub` のみを選択します（↓ キーとスペース キーを使用）。

    ```console
    ? オプションを選択してください 確認してアプリの初期化を続行
    デフォルトでは、サービスは実行されている Azure Container Apps 環境の内部からのみ到達可能です。ここでサービスを選択すると、インターネットからも到達可能になります。
    ? インターネットに公開するサービスを選択してください  [矢印で移動、スペースで選択、<右>ですべて、<左>でなし、タイプでフィルター]
          [ ]  api
        > [x]  myweatherhub
    ```

1. 最後に、環境名を指定します。これは Azure でプロビジョニングされるリソースの命名と、`dev` や `prod` などの異なる環境の管理に使用されます。

    ```console
    Azure でアプリを実行するためのファイル生成：
    
      (✓) 完了： ./azure.yaml の生成
      (✓) 完了： ./next-steps.md の生成
    
    成功： アプリはクラウド対応です！
    このディレクトリで azd up コマンドを実行して、Azure でアプリをプロビジョニングしてデプロイできます。アプリの構成の詳細については、./next-steps.md を参照してください
    ```

`azd` は多数のファイルを生成し、作業ディレクトリに配置します。これらのファイルは：

- _azure.yaml_：.NET Aspire AppHost プロジェクトなどのアプリのサービスを記述し、Azure リソースにマップします。
- _.azure/config.json_：現在のアクティブな環境が何かを `azd` に通知する構成ファイル。
- _.azure/aspireazddev/.env_：環境固有のオーバーライドを含みます。
- _.azure/aspireazddev/config.json_：この環境でパブリック エンドポイントを持つべきサービスを `azd` に通知する構成ファイル。

[](https://learn.microsoft.com/dotnet/aspire/deployment/azure/aca-deployment?tabs=visual-studio%2Cinstall-az-windows%2Cpowershell&pivots=azure-azd#deploy-the-app)

### アプリのデプロイ

`azd` が初期化されると、プロビジョニングとデプロイのプロセスを単一のコマンドとして実行できます。[azd up](https://learn.microsoft.com/azure/developer/azure-developer-cli/reference#azd-up)

```console
デフォルトでは、サービスは実行されている Azure Container Apps 環境の内部からのみ到達可能です。ここでサービスを選択すると、インターネットからも到達可能になります。
? インターネットに公開するサービスを選択してください webfrontend
? 使用する Azure サブスクリプションを選択してください：  1. <あなたのサブスクリプション>
? 使用する Azure の場所を選択してください： 1. <あなたの場所>

サービスのパッケージ化 (azd package)


成功： アプリケーションは 1 秒未満で Azure 用にパッケージ化されました。

Azure リソースのプロビジョニング (azd provision)
Azure リソースのプロビジョニングには時間がかかる場合があります。

サブスクリプション： <あなたのサブスクリプション>
場所： <あなたの場所>

  Azure ポータルで詳細な進行状況を確認できます：
<デプロイへのリンク>

  (✓) 完了： リソース グループ： <あなたのリソース グループ>
  (✓) 完了： Container Registry： <ID>
  (✓) 完了： Log Analytics ワークスペース： <ID>
  (✓) 完了： Container Apps 環境： <ID>
  (✓) 完了： Container App： <ID>

成功： アプリケーションは 1 分 13 秒で Azure にプロビジョニングされました。
Azure ポータルでリソース グループ <あなたのリソース グループ> の下に作成されたリソースを確認できます：
<リソース グループ概要へのリンク>

サービスのデプロイ (azd deploy)

  (✓) 完了： サービス api のデプロイ
  - エンドポイント： <内部のみ>

  (✓) 完了： サービス myweatherhub のデプロイ
  - エンドポイント： <あなたの固有の myweatherhub アプリ>.azurecontainerapps.io/


成功： アプリケーションは 1 分 39 秒で Azure にデプロイされました。
Azure ポータルでリソース グループ <あなたのリソース グループ> の下に作成されたリソースを確認できます：
<リソース グループ概要へのリンク>

成功： Azure でのプロビジョニングとデプロイの up ワークフローが 3 分 50 秒で完了しました。
```

まず、プロジェクトは `azd package` フェーズでコンテナーにパッケージ化され、その後 `azd provision` フェーズでアプリが必要とするすべての Azure リソースがプロビジョニングされます。

`provision` が完了すると、`azd deploy` が実行されます。このフェーズでは、プロジェクトがコンテナーとして Azure Container Registry インスタンスにプッシュされ、その後コードがホストされる Azure Container Apps の新しいリビジョンを作成するために使用されます。

この時点で、アプリがデプロイされて構成され、Azure ポータルを開いてリソースを探索できます。

## リソースのクリーンアップ

作成した Azure リソースが不要になったら、次の Azure CLI コマンドを実行してリソース グループを削除します。リソース グループを削除すると、その中に含まれるリソースも削除されます。

```console
az group delete --name <あなたのリソース グループ名>
```

**次**: [モジュール #10: 高度なコンテナー管理](10-container-management.md)

  > [!TIP]
  > .NET Aspire の公開には、最新バージョンの `azd` CLI が必要です。これは .NET Aspire ワークロードと一緒にインストールされるはずですが、CLI がインストールされていないもしくは、最新でないという通知が表示された場合は、このチュートリアルの次の部分の指示に従ってインストールできます。

1. 発行対象として **Azure Container Apps for .NET Aspire** を選択します。
    ![発行ダイアログのワークフローのスクリーンショット](./../../media/vs-deploy.png)
1. **AzDev Environment** ステップで、希望する **Subscription** と **Location** の値を選択し、**Environment name** に aspire-weather などの名前を入力します。環境名は Azure Container Apps 環境リソースの名前付けを決定します。
1. **Finish** を選択して環境を作成し、**Close** を選択してダイアログ ワークフローを終了し、デプロイメント環境の概要を表示します。
1. **Publish** を選択して、Azure 上のリソースをプロビジョニングしてデプロイします。
    > [!TIP]
    > このプロセスは完了までに数分かかる場合があります。Visual Studio はデプロイメントの進行状況を出力ログで提供し、これらの更新を見ながら公開の仕組みについて多くのことを学ぶことができます。このプロセスには、リソースグループ、Azure Container Registry、Log Analytics ワークスペース、および Azure Container Apps 環境の作成が含まれます。アプリはその後、Azure Container Apps 環境にデプロイされます。

1. 公開が完了すると、Visual Studio は環境画面の下部にリソース URL を表示します。これらのリンクを使用して、デプロイされたさまざまなリソースを表示します。**webfrontend URL** を選択して、デプロイされたアプリをブラウザで開きます。
    ![公開プロセスとデプロイされたリソースのスクリーンショット](./../../media/vs-publish-complete.png)

## Azure Developer CLI のインストール

`azd` のインストール手順はオペレーティングシステムによって異なりますが、`winget`、`brew`、`apt`、または直接 `curl` を介して広く利用できます。`azd` をインストールするには、[Azure Developer CLI のインストール](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd) を参照してください。

### テンプレートの初期化

1. 新しいターミナルウィンドウを開き、.NET Aspire プロジェクトのルートに `cd` します。
1. `azd init` コマンドを実行してプロジェクトを `azd` で初期化し、ローカルディレクトリ構造を検査してアプリの種類を判断します。

    ```console
    azd init
    ```

    `azd init` コマンドの詳細については、[azd init](https://learn.microsoft.com/azure/developer/azure-developer-cli/reference#azd-init) を参照してください。
1. アプリを初めて初期化する場合、`azd` は環境名を尋ねます。

    ```console
    Initializing an app to run on Azure (azd init)
    
    ? Enter a new environment name: [? for help]
    ```

    続行するために希望する環境名を入力します。`azd` を使用して環境を管理する方法の詳細については、[azd env](https://learn.microsoft.com/azure/developer/azure-developer-cli/reference#azd-env) を参照してください。
1. `azd` が 2 つのアプリ初期化オプションを提示したときに、**現在のディレクトリのコードを使用** を選択します。

    ```console
    ? How do you want to initialize your app?  [Use arrows to move, type to filter]
    > Use code in the current directory
      Select a template
    ```

1. ディレクトリをスキャンした後、`azd` は検出された .NET Aspire AppHost プロジェクトを確認するように求めます。**Confirm and continue initializing my app** オプションを選択します。

    ```console
    Detected services:
    
      .NET (Aspire)
      Detected in: D:\source\repos\letslearn-dotnet-aspire\complete\AppHost\AppHost.csproj
    
    azd will generate the files necessary to host your app on Azure using Azure Container Apps.
    
    ? Select an option  [Use arrows to move, type to filter]
    > Confirm and continue initializing my app
      Cancel and exit
    ```

1. `azd` は、.NET Aspire ソリューション内の各プロジェクトを提示し、どのプロジェクトを HTTP イングレスがパブリックに公開されるようにデプロイするかを尋ねます。API を Azure Container Apps 環境にプライベートに保ち、パブリックに公開しないため、`myweatherhub` のみを選択します (↓ と Space キーを使用します) 。

    ```console
    ? Select an option Confirm and continue initializing my app
    By default, a service can only be reached from inside the Azure Container Apps environment it is running in. Selecting a service here will also allow it to be reached from the Internet.
    ? Select which services to expose to the Internet  [Use arrows to move, space to select, <right> to all, <left> to none, type to filter]
      [ ]  apiservice
    > [x]  myweatherhub
    ```

1. 最後に、プロビジョニングされた Azure リソースの名前付けや `dev` や `prod` などの異なる環境の管理に使用される環境名を指定します。

    ```console
    Generating files to run your app on Azure:
    
      (✓) Done: Generating ./azure.yaml
      (✓) Done: Generating ./next-steps.md
    
    SUCCESS: Your app is ready for the cloud!
    You can provision and deploy your app to Azure by running the azd up command in this directory. For more information on configuring your app, see ./next-steps.md
    ```

`azd` は、作業ディレクトリにいくつかのファイルを生成して配置します。これらのファイルは次のとおりです：

- _azure.yaml_: .NET Aspire AppHost プロジェクトなどのアプリのサービスを説明し、それらを Azure リソースにマッピングします。
- _.azure/config.json_:  azd に現在のアクティブな 環境 を通知する構成ファイル。
- _.azure/aspireazddev/.env_: 環境固有のオーバーライドを含みます。
- _.azure/aspireazddev/config.json_: この環境でパブリック エンドポイントを持つべきサービスを `azd` に通知する構成ファイル。

[.NET Aspire アプリの Azure Container Apps へのデプロイ](https://learn.microsoft.com/dotnet/aspire/deployment/azure/aca-deployment?tabs=visual-studio%2Cinstall-az-windows%2Cpowershell&pivots=azure-azd#deploy-the-app)

### アプリのデプロイ

`azd` が初期化されたら、プロビジョニングとデプロイメントプロセスは単一のコマンド [azd up](https://learn.microsoft.com/azure/developer/azure-developer-cli/reference#azd-up) で実行できます。

```console

By default, a service can only be reached from inside the Azure Container Apps environment it is running in. Selecting a service here will also allow it to be reached from the Internet.
? Select which services to expose to the Internet webfrontend
? Select an Azure Subscription to use:  1. <YOUR SUBSCRIPTION>
? Select an Azure location to use: 1. <YOUR LOCATION>

Packaging services (azd package)


SUCCESS: Your application was packaged for Azure in less than a second.

Provisioning Azure resources (azd provision)
Provisioning Azure resources can take some time.

Subscription: <YOUR SUBSCRIPTION>
Location: <YOUR LOCATION>

  You can view detailed progress in the Azure Portal:
<LINK TO DEPLOYMENT>

  (✓) Done: Resource group: <YOUR RESOURCE GROUP>
  (✓) Done: Container Registry: <ID>
  (✓) Done: Log Analytics workspace: <ID>
  (✓) Done: Container Apps Environment: <ID>
  (✓) Done: Container App: <ID>

SUCCESS: Your application was provisioned in Azure in 1 minute 13 seconds.
You can view the resources created under the resource group <YOUR RESOURCE GROUP> in Azure Portal:
<LINK TO RESOURCE GROUP OVERVIEW>

Deploying services (azd deploy)

  (✓) Done: Deploying service apiservice
  - Endpoint: <YOUR UNIQUE apiservice APP>.azurecontainerapps.io/

  (✓) Done: Deploying service webfrontend
  - Endpoint: <YOUR UNIQUE webfrontend APP>.azurecontainerapps.io/


SUCCESS: Your application was deployed to Azure in 1 minute 39 seconds.
You can view the resources created under the resource group <YOUR RESOURCE GROUP> in Azure Portal:
<LINK TO RESOURCE GROUP OVERVIEW>

SUCCESS: Your up workflow to provision and deploy to Azure completed in 3 minutes 50 seconds.
```

まず、プロジェクトは `azd package` フェーズ中にコンテナにパッケージ化され、次に `azd provision` フェーズでアプリが必要とするすべての Azure リソースがプロビジョニングされます。

`provision` が完了すると、`azd deploy` が行われます。このフェーズでは、プロジェクトはコンテナとして Azure Container Registry インスタンスにプッシュされ、その後コードをホストするために新しい Azure Container Apps のリビジョンが作成されます。

この時点でアプリがデプロイされ構成されており、Azure ポータルを開いてリソースを探索することができます。

## リソースのクリーンアップ

作成した Azure リソースが不要になったときに、次の Azure CLI コマンドを実行してリソースグループを削除します。リソースグループを削除すると、その中に含まれるリソースも削除されます。

```console
az group delete --name <your-resource-group-name>
```