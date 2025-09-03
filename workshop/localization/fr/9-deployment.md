# Déployer une application .NET Aspire sur Azure Container Apps

.NET Aspire est optimisé pour les applications destinées à s'exécuter dans des environnements conteneurisés. [Azure Container Apps](https://learn.microsoft.com/azure/container-apps/overview) est un environnement entièrement géré qui vous permet d'exécuter des microservices et des applications conteneurisées sur une plateforme sans serveur. Cet article vous guidera dans la création d'une nouvelle solution .NET Aspire et son déploiement sur Microsoft Azure Container Apps en utilisant Visual Studio et l'Azure Developer CLI (`azd`).

Dans cet exemple, nous supposerons que vous déployez l'application MyWeatherHub des sections précédentes. Vous pouvez utiliser le code que vous avez construit, ou vous pouvez utiliser le code dans le répertoire **complete**. Cependant, les étapes générales sont les mêmes pour toute application .NET Aspire.

## Déployer l'application avec Visual Studio

1. Dans l'Explorateur de solutions, cliquez avec le bouton droit sur le projet **AppHost** et sélectionnez **Publier** pour ouvrir la boîte de dialogue **Publier**.

    > Publier .NET Aspire nécessite la version actuelle de la CLI `azd`. Cela devrait être installé avec la charge de travail .NET Aspire, mais si vous recevez une notification indiquant que la CLI n'est pas installée ou à jour, vous pouvez suivre les instructions dans la partie suivante de ce tutoriel pour l'installer.

1. Sélectionnez **Azure Container Apps pour .NET Aspire** comme cible de publication.

    ![Une capture d'écran du flux de travail de la boîte de dialogue de publication.](media/vs-deploy.png)

1. À l'étape **Environnement AzDev**, sélectionnez vos valeurs **Abonnement** et **Emplacement** désirées puis entrez un **Nom d'environnement** tel que _aspire-weather_. Le nom de l'environnement détermine la nomenclature des ressources de l'environnement Azure Container Apps.
1. Sélectionnez **Terminer** pour créer l'environnement, puis **Fermer** pour quitter le flux de travail de la boîte de dialogue et voir le résumé de l'environnement de déploiement.
1. Sélectionnez **Publier** pour provisionner et déployer les ressources sur Azure.

    > Ce processus peut prendre plusieurs minutes à compléter. Visual Studio fournit des mises à jour de statut sur le progrès du déploiement dans les journaux de sortie et vous pouvez apprendre beaucoup sur le fonctionnement de la publication en observant ces mises à jour ! Vous verrez que le processus implique la création d'un groupe de ressources, d'un Azure Container Registry, d'un espace de travail Log Analytics et d'un environnement Azure Container Apps. L'application est ensuite déployée dans l'environnement Azure Container Apps.

1. Lorsque la publication se termine, Visual Studio affiche les URLs des ressources en bas de l'écran de l'environnement. Utilisez ces liens pour voir les diverses ressources déployées. Sélectionnez l'URL **webfrontend** pour ouvrir un navigateur vers l'application déployée.

    ![Une capture d'écran du processus de publication terminé et des ressources déployées.](media/vs-publish-complete.png)

## Déployer l'application avec l'Azure Developer CLI (azd)

### Installer l'Azure Developer CLI

Le processus d'installation d'`azd` varie selon votre système d'exploitation, mais il est largement disponible via `winget`, `brew`, `apt`, ou directement via `curl`. Pour installer `azd`, voir [Installer Azure Developer CLI](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd).

### Nouveau dans .NET Aspire 9.4 : commande aspire deploy

.NET Aspire 9.4 introduit la commande `aspire deploy` (aperçu/feature flag) qui étend les capacités de publication pour déployer activement vers des environnements cibles. Cette commande fournit des flux de travail de déploiement améliorés avec une logique personnalisée pré/post-déploiement.

Pour activer cette fonctionnalité :

```bash
aspire config set features.deployCommandEnabled true
```

Ensuite, vous pouvez utiliser :

```bash
aspire deploy
```

Cette commande fournit des rapports de progression améliorés, de meilleurs messages d'erreur, et prend en charge les hooks de déploiement personnalisés pour les scénarios de déploiement complexes.

### Initialiser le modèle

> Prérequis :
>
> - Assurez-vous d'être connecté : exécutez `azd login` et sélectionnez l'abonnement Azure correct.
> - Exécutez les commandes suivantes depuis le dossier qui contient votre AppHost (pour ce dépôt, typiquement le dossier `complete` si vous déployez l'échantillon terminé).

1. Ouvrez une nouvelle fenêtre de terminal et naviguez (`cd`) vers la racine de votre projet .NET Aspire.
1. Exécutez la commande `azd init` pour initialiser votre projet avec `azd`, qui inspectera la structure de répertoire locale et déterminera le type d'application.

    ```console
    azd init
    ```

    Pour plus d'informations sur la commande `azd init`, voir [azd init](https://learn.microsoft.com/azure/developer/azure-developer-cli/reference#azd-init).
1. Si c'est la première fois que vous initialisez l'application, `azd` vous demande le nom de l'environnement :

    ```console
    Initialisation d'une application pour s'exécuter sur Azure (azd init)
    
    ? Entrez un nouveau nom d'environnement : [? pour l'aide]
    ```

    Entrez le nom d'environnement désiré pour continuer. Pour plus d'informations sur la gestion des environnements avec `azd`, voir [azd env](https://learn.microsoft.com/azure/developer/azure-developer-cli/reference#azd-env).
1. Sélectionnez **Utiliser le code dans le répertoire actuel** lorsque `azd` vous propose deux options d'initialisation d'application.

    ```console
    ? Comment voulez-vous initialiser votre application ?  [Utilisez les flèches pour bouger, tapez pour filtrer]
    > Utiliser le code dans le répertoire actuel
      Sélectionner un modèle
    ```

1. Après avoir scanné le répertoire, `azd` vous demande de confirmer qu'il a trouvé le bon projet .NET Aspire _AppHost_. Sélectionnez l'option **Confirmer et continuer l'initialisation de mon application**.

    ```console
    Services détectés :
    
      .NET (Aspire)
      Détecté dans : D:\source\repos\letslearn-dotnet-aspire\complete\AppHost\AppHost.csproj
    
    azd générera les fichiers nécessaires pour héberger votre application sur Azure en utilisant Azure Container Apps.
    
    ? Sélectionnez une option  [Utilisez les flèches pour bouger, tapez pour filtrer]
    > Confirmer et continuer l'initialisation de mon application
      Annuler et quitter
    ```

1. `azd` présente chacun des projets de la solution .NET Aspire et vous demande d'identifier lesquels déployer avec l'ingress HTTP ouvert publiquement à tout le trafic internet. Sélectionnez seulement `myweatherhub` (en utilisant les touches ↓ et Espace), puisque vous voulez que l'API (`api`) soit privée à l'environnement Azure Container Apps et non disponible publiquement.

    ```console
    ? Sélectionnez une option Confirmer et continuer l'initialisation de mon application
    Par défaut, un service ne peut être atteint que depuis l'intérieur de l'environnement Azure Container Apps dans lequel il s'exécute. Sélectionner un service ici permettra aussi qu'il soit atteint depuis Internet.
    ? Sélectionnez quels services exposer à Internet  [Utilisez les flèches pour bouger, espace pour sélectionner, <droite> pour tous, <gauche> pour aucun, tapez pour filtrer]
          [ ]  api
        > [x]  myweatherhub
    ```

1. Enfin, spécifiez le nom de l'environnement, qui est utilisé pour nommer les ressources provisionnées dans Azure et gérer différents environnements tels que `dev` et `prod`.

    ```console
    Génération de fichiers pour exécuter votre application sur Azure :
    
      (✓) Terminé : Génération de ./azure.yaml
      (✓) Terminé : Génération de ./next-steps.md
    
    SUCCÈS : Votre application est prête pour le cloud !
    Vous pouvez provisionner et déployer votre application sur Azure en exécutant la commande azd up dans ce répertoire. Pour plus d'informations sur la configuration de votre application, voir ./next-steps.md
    ```

`azd` génère un certain nombre de fichiers et les place dans le répertoire de travail. Ces fichiers sont :

- _azure.yaml_ : Décrit les services de l'application, tels que le projet .NET Aspire AppHost, et les mappe aux ressources Azure.
- _.azure/config.json_ : Fichier de configuration qui informe `azd` de quel est l'environnement actif actuel.
- _.azure/aspireazddev/.env_ : Contient les remplacements spécifiques à l'environnement.
- _.azure/aspireazddev/config.json_ : Fichier de configuration qui informe `azd` quels services doivent avoir un point de terminaison public dans cet environnement.

[](https://learn.microsoft.com/dotnet/aspire/deployment/azure/aca-deployment?tabs=visual-studio%2Cinstall-az-windows%2Cpowershell&pivots=azure-azd#deploy-the-app)

### Déployer l'application

Une fois qu'`azd` est initialisé, le processus de provisionnement et de déploiement peut être exécuté comme une seule commande, [azd up](https://learn.microsoft.com/azure/developer/azure-developer-cli/reference#azd-up).

```console
Par défaut, un service ne peut être atteint que depuis l'intérieur de l'environnement Azure Container Apps dans lequel il s'exécute. Sélectionner un service ici permettra aussi qu'il soit atteint depuis Internet.
? Sélectionnez quels services exposer à Internet webfrontend
? Sélectionnez un abonnement Azure à utiliser :  1. <VOTRE ABONNEMENT>
? Sélectionnez un emplacement Azure à utiliser : 1. <VOTRE EMPLACEMENT>

Empaquetage des services (azd package)


SUCCÈS : Votre application a été empaquetée pour Azure en moins d'une seconde.

Provisionnement des ressources Azure (azd provision)
Le provisionnement des ressources Azure peut prendre du temps.

Abonnement : <VOTRE ABONNEMENT>
Emplacement : <VOTRE EMPLACEMENT>

  Vous pouvez voir les progrès détaillés dans le portail Azure :
<LIEN VERS LE DÉPLOIEMENT>

  (✓) Terminé : Groupe de ressources : <VOTRE GROUPE DE RESSOURCES>
  (✓) Terminé : Container Registry : <ID>
  (✓) Terminé : Espace de travail Log Analytics : <ID>
  (✓) Terminé : Environnement Container Apps : <ID>
  (✓) Terminé : Container App : <ID>

SUCCÈS : Votre application a été provisionnée dans Azure en 1 minute 13 secondes.
Vous pouvez voir les ressources créées sous le groupe de ressources <VOTRE GROUPE DE RESSOURCES> dans le portail Azure :
<LIEN VERS L'APERÇU DU GROUPE DE RESSOURCES>

Déploiement des services (azd deploy)

  (✓) Terminé : Déploiement du service api
  - Point de terminaison : <interne seulement>

  (✓) Terminé : Déploiement du service myweatherhub
  - Point de terminaison : <VOTRE APPLICATION myweatherhub UNIQUE>.azurecontainerapps.io/


SUCCÈS : Votre application a été déployée sur Azure en 1 minute 39 secondes.
Vous pouvez voir les ressources créées sous le groupe de ressources <VOTRE GROUPE DE RESSOURCES> dans le portail Azure :
<LIEN VERS L'APERÇU DU GROUPE DE RESSOURCES>

SUCCÈS : Votre flux de travail up pour provisionner et déployer sur Azure s'est terminé en 3 minutes 50 secondes.
```

D'abord, les projets seront empaquetés en conteneurs pendant la phase `azd package`, suivie par la phase `azd provision` pendant laquelle toutes les ressources Azure dont l'application aura besoin sont provisionnées.

Une fois que `provision` est terminé, `azd deploy` aura lieu. Pendant cette phase, les projets sont poussés comme conteneurs dans une instance Azure Container Registry, puis utilisés pour créer de nouvelles révisions d'Azure Container Apps dans lesquelles le code sera hébergé.

À ce point, l'application a été déployée et configurée, et vous pouvez ouvrir le portail Azure et explorer les ressources.

## Nettoyer les ressources

Exécutez la commande Azure CLI suivante pour supprimer le groupe de ressources lorsque vous n'avez plus besoin des ressources Azure que vous avez créées. Supprimer le groupe de ressources supprime également les ressources qu'il contient.

```console
az group delete --name <nom-de-votre-groupe-de-ressources>
```

**Suivant** : [Module #10 : Gestion Avancée de Conteneurs](10-container-management.md)

Dans cet exemple, nous supposerons que vous déployez l'application MyWeatherHub des sections précédentes. Vous pouvez utiliser le code que vous avez créé ou le code contenu dans le répertoire **complete**. Cependant, les étapes sont les mêmes pour n’importe quelle application .NET Aspire.

## Déployer l'application avec Visual Studio

1. Dans l'explorateur de solutions, cliquez avec le bouton droit sur le projet **AppHost** et sélectionnez **Publier** pour ouvrir la boîte de dialogue **Publier**.

 > [!TIP]
 > La publication .NET Aspire nécessite la version actuelle de la CLI `azd`. Celui-ci doit être installé avec le "workload" .NET Aspire, mais si vous recevez une notification indiquant que la CLI n'est pas installée ou à jour, vous pouvez suivre les instructions de la partie suivante de ce didacticiel pour l'installer.

1. Sélectionnez **Azure Container Apps for .NET Aspire** comme cible de publication.
 ![Une capture d'écran du flux de travail de la boîte de dialogue de publication.](./../../media/vs-deploy.png)
1. À l'étape **AzDev Environment**, sélectionnez les valeurs **Subscription** et **Location** souhaitées, puis entrez un **Environment name** tel que _aspire-weather_. Le nom de l’environnement détermine le nom des ressources de l’environnement Azure Container Apps.
1. Sélectionnez **Finish** pour créer l'environnement, puis **Close** pour quitter le workflow de boîte de dialogue et afficher le résumé de l'environnement de déploiement.
1. Sélectionnez **Publish** pour provisionner et déployer les ressources sur Azure.

 > [!TIP]
 > Ce processus peut prendre plusieurs minutes. Visual Studio fournit des mises à jour d'état sur la progression du déploiement dans les journaux de sortie et vous pouvez en apprendre beaucoup sur le fonctionnement de la publication en regardant ces mises à jour! Vous verrez que le processus implique la création d’un groupe de ressources, d’un Azure Container Registry, d’un espace de travail Log Analytics et d’un environnement Azure Container Apps. L’application est ensuite déployée dans l’environnement Azure Container Apps.

1. Une fois la publication terminée, Visual Studio affiche les URL des ressources en bas de l'écran de l'environnement. Utilisez ces liens pour afficher les différentes ressources déployées. Sélectionnez l'URL **webfrontend** pour ouvrir un navigateur sur l'application déployée.
    ![Une capture d'écran du processus de publication terminé et des ressources déployées.](./../../media/vs-publish-complete.png)

## Installer la CLI du développeur Azure

Le processus d'installation de `azd` varie en fonction de votre système d'exploitation, mais il est largement disponible via «winget», «brew», «apt» ou directement via «curl». Pour installer `azd`, consultez [Installer Azure Developer CLI](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd).


### Initialiser le modèle

1. Ouvrez une nouvelle fenêtre de terminal et `cd` à la racine de votre projet .NET Aspire.
1. Exécutez la commande `azd init` pour initialiser votre projet avec `azd`, qui inspectera la structure des répertoires locaux et déterminera le type d'application.

    ```console
    azd init
    ```

    Pour plus d'informations sur la commande `azd init`, consultez [azd init](https://learn.microsoft.com/azure/developer/azure-developer-cli/reference#azd-init).
1. Si c'est la première fois que vous initialisez l'application, `azd` vous demande le nom de l'environnement :

    ```console
    Initializing an app to run on Azure (azd init)
    
    ? Enter a new environment name: [? for help]
    ```

    Entrez le nom de l'environnement souhaité pour continuer. Pour plus d'informations sur la gestion des environnements avec `azd`, consultez [azd env](https://learn.microsoft.com/azure/developer/azure-developer-cli/reference#azd-env).
1. Sélectionnez **Use code in the current directory** lorsque `azd` vous propose deux options d'initialisation de l'application.

    ```console
    ? How do you want to initialize your app?  [Use arrows to move, type to filter]
    > Use code in the current directory
      Select a template
    ```

1. Après avoir analysé le répertoire, `azd` vous invite à confirmer qu'il a trouvé le bon projet .NET Aspire _AppHost_. Sélectionnez l'option **Confirmer et continuer l'initialisation de mon application**.

    ```console
    Detected services:
    
      .NET (Aspire)
      Detected in: D:\source\repos\letslearn-dotnet-aspire\complete\AppHost\AppHost.csproj
    
    azd will generate the files necessary to host your app on Azure using Azure Container Apps.
    
    ? Select an option  [Use arrows to move, type to filter]
    > Confirm and continue initializing my app
      Cancel and exit
    ```

1. `azd` présente chacun des projets de la solution .NET Aspire et vous invite à identifier lesquels déployer avec une entrée HTTP ouverte publiquement à tout le trafic Internet. Sélectionnez uniquement `myweatherhub` (à l'aide des touches ↓ et Espace), car vous souhaitez que l'API soit privée pour l'environnement Azure Container Apps et _not_ disponible publiquement.

    ```console
    ? Select an option Confirm and continue initializing my app
    By default, a service can only be reached from inside the Azure Container Apps environment it is running in. Selecting a service here will also allow it to be reached from the Internet.
    ? Select which services to expose to the Internet  [Use arrows to move, space to select, <right> to all, <left> to none, type to filter]
      [ ]  apiservice
    > [x]  myweatherhub
    ```

1. Enfin, spécifiez le nom de l'environnement, qui est utilisé pour nommer les ressources provisionnées dans Azure et gérer différents environnements tels que `dev` et `prod`.

    ```console
    Generating files to run your app on Azure:
    
      (✓) Done: Generating ./azure.yaml
      (✓) Done: Generating ./next-steps.md
    
    SUCCESS: Your app is ready for the cloud!
    You can provision and deploy your app to Azure by running the azd up command in this directory. For more information on configuring your app, see ./next-steps.md
    ```

`azd` génère un certain nombre de fichiers et les place dans le répertoire de travail. Ces fichiers sont :

- _azure.yaml_ : décrit les services de l'application, tels que le projet .NET Aspire AppHost, et les mappe aux ressources Azure.
- _.azure/config.json_ : fichier de configuration qui informe `azd` quel est l'environnement actif actuel.
- _.azure/aspireazddev/.env_ : contient des remplacements spécifiques à l'environnement.
- _.azure/aspireazddev/config.json_ : fichier de configuration qui informe `azd` quels services doivent avoir un point de terminaison public dans cet environnement.

[](https://learn.microsoft.com/dotnet/aspire/deployment/azure/aca-deployment?tabs=visual-studio%2Cinstall-az-windows%2Cpowershell&pivots=azure-azd#deploy-the-app)

### Déployer l'application

Une fois `azd` initialisé, le processus de provisionnement et de déploiement peut être exécuté en tant que commande unique, [azd up](https://learn.microsoft.com/azure/developer/azure-developer-cli/reference#azd-up ).

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

Tout d'abord, les projets seront regroupés dans des conteneurs pendant la phase `package azd`, suivie de la phase `azd provision` au cours de laquelle toutes les ressources Azure dont l'application aura besoin sont provisionnées.

Une fois le `provision` terminé, le `azd deploy` aura lieu. Au cours de cette phase, les projets sont poussés en tant que conteneurs dans une instance Azure Container Registry, puis utilisés pour créer de nouvelles révisions d'Azure Container Apps dans lesquelles le code sera hébergé.

À ce stade, l’application a été déployée et configurée, et vous pouvez ouvrir le portail Azure et explorer les ressources.

## Nettoyer les ressources

Exécutez la commande Azure CLI suivante pour supprimer le groupe de ressources lorsque vous n’avez plus besoin des ressources Azure que vous avez créées. La suppression du groupe de ressources supprime également les ressources qu'il contient.

```console
az group delete --name <votre-nom-de-groupe-de-ressources>
```