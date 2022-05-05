---
ms.openlocfilehash: 07a00d0079982d7fe6ec4ba16e3d6d06420b0190
ms.sourcegitcommit: 3e65a56f5041a2106e99b6fe06be5dc7a931198e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/22/2022
ms.locfileid: "144385230"
---

# <a name="modern-data-warehouse-openhack"></a>Un entrepôt de données moderne - OpenHack

## <a name="setting-up-permissions"></a>Configuration des autorisations  

Avant de continuer, veillez à bien comprendre les autorisations nécessaires pour exécuter l’OpenHack sur votre abonnement Azure.

Les participants doivent disposer d’autorisations à un abonnement Azure qui permettent la création de ressources dans leur groupe de ressources. En outre, les participants doivent disposer d’autorisations à un abonnement suffisantes pour créer des principaux de service dans Azure AD et inscrire des applications dans Azure AD. En règle générale, tout ce qui est requis est un compte d’utilisateur avec le rôle `Owner` sur leur groupe de ressources.

## <a name="common-azure-resources"></a>Ressources Azure courantes

La liste suivante répertorie les ressources Azure courantes qui sont déployées et utilisées pendant l’OpenHack. 

Assurez-vous que ces services ne sont pas bloqués par Azure Policy. Comme il s’agit d’un OpenHack, les services que les participants peuvent utiliser ne sont pas limités à cette liste, donc les abonnements avec un catalogue de services étroitement contrôlé peuvent rencontrer des problèmes si le service qu’un participant souhaite utiliser est désactivé via une stratégie.

| Ressource Azure           | Fournisseurs de ressources |
| ------------------------ | --------------------------------------- |
| Azure Cosmos DB          | Microsoft.DocumentDB                    | 
| Azure Data Factory       | Microsoft.DataFactory                   |
| Azure Purview            | Microsoft.Purview                       |
| Azure Synapse            | Microsoft.Synapse                       |
| Azure Databricks         | Microsoft.Databricks                    |
| Azure SQL Database       | Microsoft.SQL                           |
| Stockage Azure            | Microsoft.Storage                       |
| Azure Data Lake Store    | Microsoft.DataLakeStore                 |
| Machines virtuelles Azure   | Microsoft.Compute                       |

> Remarque :  Inscription du fournisseur de ressources est accessible sous https://portal.azure.com/_nom_de_votre_locataire_.onmicrosoft.com/resource/subscriptions/_ID_de_votre_abonnement_/resourceproviders

## <a name="attendee-computers"></a>Ordinateurs des participants

Les participants sont tenus d’installer les logiciels sur les stations de travail sur lesquelles ils font l’OpenHack. Assurez-vous qu’ils disposent des autorisations appropriées pour effectuer l’installation des logiciels.

## <a name="deployment-instructions"></a>Instructions de déploiement  

1. Ouvrez une fenêtre **PowerShell 7**, exécutez la commande suivante, si vous y êtes invité, cliquez sur **Oui pour tout** :

   ```PowerShell
   Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
   ```

2. Exécutez ce qui suit pour vous connecter au compte Azure disposant de l’attribution de rôle **Propriétaire** dans votre abonnement.

    ```PowerShell
    Connect-AzAccount
    ```

3. Si vous avez plusieurs abonnements, veillez à sélectionner l’abonnement approprié avant la prochaine étape. Utilisez `Get-AzSubscription` pour lister vos abonnements, puis utilisez la commande ci-dessous pour définir l’abonnement que vous utilisez :

    Lister les abonnements :  

    ```powershell
    Get-AzSubscription
    ```  

    Sélectionner l’abonnement à utiliser :

    ```powershell
    Select-AzSubscription -Subscription <The selected Subscription Id>
    ```

4. Attribuez les variables `$sqlpwd` et `$vmpwd` dans votre session PowerShell sous forme de **chaînes sécurisées**. Veillez à utiliser un mot de passe fort pour les deux. Suivez[ce lien](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm) pour voir les exigences de mot de passe de la machine virtuelle et [ce lien](https://docs.microsoft.com/en-us/sql/relational-databases/security/password-policy?view=sql-server-2017#password-complexity) pour SQL Server.

    ```powershell
    $PlainPassword = "demo@pass123"
    $SqlAdminLoginPassword = $PlainPassword | ConvertTo-SecureString -AsPlainText -Force
    $VMAdminPassword = $PlainPassword | ConvertTo-SecureString -AsPlainText -Force
    ```  

5. Si ce n’est déjà fait, vous devez télécharger le dossier `modern-data-warehousing` à partir du référentiel.  Vous pouvez utiliser la commande suivante pour cloner le référentiel dans le répertoire actif :

   ```shell
   git clone https://github.com/microsoft/OpenHack.git
   ```
  
6. Exécutez ce qui suit à partir du répertoire `modern-data-warehousing` du clone du référentiel OpenHack pour déployer l’environnement (ce processus peut prendre 10 à 15 minutes) :

    ```powershell
     .\BYOS-deployAll.ps1 -SqlAdminLoginPassword $SqlAdminLoginPassword -VMAdminPassword $VMAdminPassword 
    ```

### <a name="manual-step---assigning-users-to-each-resource-group"></a>Étape manuelle : affectation d’utilisateurs à chaque groupe de ressources  

Après le déploiement, ajoutez manuellement les utilisateurs appropriés avec un accès Propriétaire sur le groupe de ressources approprié à leur équipe.  

Pour obtenir des instructions détaillées sur l’affectation d’utilisateurs à des rôles, consultez ce qui suit.

[Ajouter ou supprimer des attributions de rôles Azure avec le portail Azure](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-portal)

## <a name="validate"></a>Valider  

Les groupes de ressources existent pour chacune des équipes, les membres se trouvent dans chaque équipe de manière appropriée avec l’autorisation Propriétaire sur le groupe de ressources.

Passez en revue le fichier lisez-moi pour connaître les prérequis et les autres éléments à vérifier pour chaque équipe.

## <a name="more-details-on-the-usage-of-the-services"></a>Plus de détails sur l’utilisation des services

Lorsque vous démarrez le processus, un certain nombre de modèles sont déployés.

### <a name="deploymdwopenhacklabjson"></a>DeployMDWOpenHackLab.json

Vous avez lancé cela (probablement à partir du script BYOS-deployAll.ps1).  

Ce modèle de déploiement déclenche le modèle d’orchestrateur portant le même nom qui se trouve dans le stockage Azure.  

#### <a name="parameters"></a>Paramètres

Notez que l’utilisation de `DeployMDWOpenHackLab.parameters.json` fournira la plupart de ces paramètres pour vous. Ceux nécessitant une attention au moment du déploiement ont été marqués en **gras** ci-dessous.

- SqlAdminLogin : Nom d’utilisateur de l’administrateur SQL pour **toutes** les bases de données SQL dans ce déploiement.
- **SqlAdminLoginPassword** : Mot de passe pour SqlAdminLogin sur **toutes** les bases de données SQL dans ce déploiement.
- SalesDacPacPath : URI vers le bacpac importé dans la base de données CloudSales (southridge).
- StreamingDacPacPath : URI vers le bacpac importé dans la base de données CloudStreaming (southridge).
- VMAdminUsername : Nom d’utilisateur de l’administrateur pour **toutes** les machines virtuelles dans ce déploiement
- **VMAdminPassword** : Mot de passe de l’administrateur sur **toutes** les machines virtuelles dans ce déploiement
- RentalsBackupStorageAccountName : Compte de stockage où sont stockées les sauvegardes dbbackups
- RentalsBackupFileName : Nom du fichier .bak importé dans la base de données Rentals de la machine virtuelle « locale » (VanArsdel Ltd.)
- RentalsDatabaseName : Nom de la base de données « locale » restaurée sur la machine virtuelle
- RentalsCsvFolderName : Nom du dossier contenant toutes les données CSV, dans le conteneur dbbackups
- CatalogJsonFileName : Nom du fichier JSON contenant tout le catalogue de films de Southridge Video
- SQLFictitiousCompanyNamePrefix : Nom fictif de la société utilisant la machine virtuelle SQL « locale » (VanArsdel Ltd.)
- CsvFictitiousCompanyNamePrefix : Nom fictif de la société utilisant les fichiers CSV « locaux » (Fourth Coffee)
- CloudFictitiousCompanyNamePrefix : Nom fictif de la société utilisant les bases de données cloud (Southridge)
- **location** : Région Azure cible

> Remarque : Vous pouvez exécuter le script sans aucun paramètre, car tous les scripts sont définis avec des valeurs par défaut pour les éléments critiques du déploiement.  L’utilisation de paramètres vous permet d’effectuer un remplacement en cas de besoin (par exemple, le mot de passe).  

### <a name="deploycosmosdbjson"></a>DeployCosmosDB.json

Modèle limité au provisionnement d’un compte Cosmos DB.

> La collection Cosmos DB est créée et remplie par une extension de machine virtuelle dans DeployFileVM.

#### <a name="parameters"></a>Paramètres  

- location : Région Azure cible
- namePrefix : Le compte Cosmos DB sera créé avec un nom au format `{namePrefix}-catalog-{uniqueStringForResourceGroup}`

### <a name="deployfilevmjson"></a>DeployFileVM.json

Modèle limité à la création de la machine virtuelle où une société fictive stocke ses données CSV.

> Ce déploiement de machine virtuelle contient l’extension qui télécharge non seulement les données CSV sur la machine virtuelle, mais remplit également le catalogue de films Cosmos DB.

#### <a name="parameters"></a>Paramètres

- adminUsername : Nom d’utilisateur de l’administrateur de la machine virtuelle
- adminPassword : Mot de passe de l’administrateur de la machine virtuelle
- BackupStorageAccountName : Compte de stockage contenant les données CSV
- BackupStorageContainerName : Conteneur dans le compte de stockage contenant les données CSV
- RentalsCsvFolderName : Dossier dans le conteneur contenant les données CSV
- catalogJsonFileName : Nom de fichier du catalogue de films southridge à charger dans Cosmos
- location : Région Azure cible
- namePrefix : Les ressources de la machine virtuelle utiliseront ce nom, par exemple, `{namePrefix}VM`, `{namePrefix}-PIP`, etc.

### <a name="deploysqlazurejson"></a>DeploySQLAzure.json

Ce modèle déploie deux bases de données Azure SQL dans un seul serveur et effectue une importation bacpac pour chacune.

#### <a name="parameters"></a>Paramètres

- AdminLogin : Identifiant de connexion de l’administrateur SQL
- AdminLoginPassword : Mot de passe de l’administrateur SQL
- SalesDacPacPath : Chemin vers le bacpac CloudSales
- StreamingDacPacPath : Chemin vers le bacpac CloudStreaming
- DacPacContainerSAS : Signature d’accès partagé (SAS) pour le conteneur dbbackups
- location : Région Azure cible
- namePrefix : Le nom du serveur SQL sera au format `{namePrefix-sqlserver-uniqueStringForResourceGroup}`

### <a name="deploysqlvmjson"></a>DeploySQLVM.json

Ce modèle déploie une machine virtuelle avec le serveur SQL et importe un bak pour la base de données Rentals locale.

#### <a name="parameters"></a>Paramètres

- adminUsername : Nom d’utilisateur de l’administrateur de la machine virtuelle
- adminPassword : Mot de passe de l’administrateur de la machine virtuelle
- sqlAuthenticationLogin : Nom d’utilisateur SQL
- sqlAuthenticationPassword : Mot de passe SQL
- BackupStorageAccountName : Compte de stockage contenant le bak
- BackupStorageContainerName : Conteneur dans le compte de stockage contenant le bak
- BackupFileName : Nom de fichier du bak
- DatabaseName : Nom de la base de données restaurée
- location : Région Azure cible
- namePrefix : Les ressources de la machine virtuelle utiliseront ce nom, par exemple, `{namePrefix}VM`, `{namePrefix}-PIP`, etc.
