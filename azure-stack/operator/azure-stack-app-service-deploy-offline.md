---
title: Deploy App Service in an offline environment in Azure Stack | Microsoft Docs
description: Learn how to deploy App Service in an offline Azure Stack environment secured by AD FS.
services: azure-stack
documentationcenter: ''
author: BryanLa
manager: femila
editor:

ms.assetid:
ms.service: azure-stack
ms.workload: app-service
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/29/2019
ms.author: anwestg
ms.reviewer: anwestg
ms.lastreviewed: 01/11/2019

---
# Deploy App Service in an offline environment in Azure Stack

*Applies to: Azure Stack integrated systems and Azure Stack Development Kit*

> [!IMPORTANT]
> Apply the 1907 update to your Azure Stack integrated system or deploy the latest Azure Stack Development Kit (ASDK) before you deploy Azure App Service 1.7.

By following the instructions in this article, you can deploy the [App Service resource provider](azure-stack-app-service-overview.md) to an Azure Stack environment that is:

- Not connected to the internet.
- Secured by Active Directory Federation Services (AD FS).

> [!IMPORTANT]
> Before you run the resource provider installer, make sure you've completed the steps in [Prerequisites for deploying App Service on Azure Stack](azure-stack-app-service-before-you-get-started.md). You should also read the [release notes](azure-stack-app-service-release-notes-update-seven.md) that accompany the 1.7 release to learn about new functionality, fixes, and any known issues that could affect your deployment.

To add the App Service resource provider to your offline Azure Stack deployment, you must complete these top-level tasks:

1. Complete the [prerequisite steps](azure-stack-app-service-before-you-get-started.md) (like purchasing certificates, which can take a few days to receive).
2. [Download and extract the installation and helper files](azure-stack-app-service-before-you-get-started.md) to a machine that's connected to the internet.
3. Create an offline installation package.
4. Run the appservice.exe installer file.

## Create an offline installation package

To deploy App Service in an offline environment, first create an offline installation package on a machine that's connected to the internet.

1. Run the AppService.exe installer on a machine that's connected to the internet.

2. Select **Advanced** > **Create offline installation package**.

    ![Create an offline package in App Service Installer][1]

3. The App Service installer creates an offline installation package and displays the path to it. You can select **Open folder** to open the folder in File Explorer.

    ![Offline installation package generated successfully in App Service Installer](media/azure-stack-app-service-deploy-offline/image02.png)

4. Copy the installer (AppService.exe) and the offline installation package to your Azure Stack host machine.

## Complete the offline installation of App Service on Azure Stack

1. Run appservice.exe as an admin from a computer that can reach the Azure Stack Admin Azure Resource Management endpoint.

1. Select **Advanced** > **Complete offline installation**.

    ![Complete offline installation in App Service Installer][2]

1. Browse to the location of the offline installation package you previously created, and then select **Next**.

    ![Specify offline installation package path im App Service Installer](media/azure-stack-app-service-deploy-offline/image04.png)

1. Review and accept the Microsoft Software License Terms, and then select **Next**.

1. Review and accept the third-party license terms, and then select **Next**.

1. Make sure the App Service cloud configuration info is correct. If you used the default settings during ASDK deployment, you can accept the default values here. However, if you customized the options when you deployed Azure Stack or are deploying on an integrated system, you must edit the values in this window to reflect those changes. For example, if you use the domain suffix mycloud.com, your Azure Stack Tenant Azure Resource Manager endpoint must change to `management.<region>.mycloud.com`. After you confirm your info, select **Next**.

    ![Configure Azure App Service cloud in App Service Installer][3]

1. On the next page:

   a. Select the **Connect** button next to the **Azure Stack Subscriptions** box. 

   b. Provide your admin account. For example: cloudadmin@azurestack.local. Enter your password and then select **Sign In**.

   c. In the **Azure Stack Subscriptions** box, select **Default Provider Subscription**.
   > [!NOTE]
   > App Service can be deployed only into **Default Provider Subscription**.

   d. In the **Azure Stack Locations** box, select the location that corresponds to the region you're deploying to. For example, if you're deploying to the ASDK, select **local**.

   e. Select **Next**.

      ![Azure Stack subscriptions and locations in App Service Installer][4]

1. You can allow the App Service installer to create a virtual network and associated subnets. Or, you can deploy into an existing virtual network, as configured through [these steps](azure-stack-app-service-before-you-get-started.md#virtual-network).
   - To use the App Service installer method, select **Create VNet with default settings**, accept the defaults, and then select **Next**.
   - To deploy into an existing network, select **Use existing VNet and Subnets**, and then:
       1. Select the **Resource Group** option that contains your virtual network.
       2. Choose the **Virtual Network** name you want to deploy into.
       3. Select the correct **Subnet** values for each of the required role subnets.
       4. Select **Next**.

      ![Virtual network and subnet info in App Service Installer][5]

1. Enter the info for your file share and then select **Next**. The address of the file share must use the Fully Qualified Domain Name (FQDN) or IP address of your file server. For example: \\\appservicefileserver.local.cloudapp.azurestack.external\websites, or \\\10.0.0.1\websites.  If you're using a file server that's domain-joined, you must provide the full user name, including the domain. For example: `<myfileserverdomain>\<FileShareOwner>`.

    > [!NOTE]
    > The installer tries to test connectivity to the file share before proceeding. However, if you choose to deploy into an existing virtual network, the installer might be unable to connect to the file share and displays a warning asking whether you want to continue. Verify the file share info and continue if it's correct.

   ![File share info in App Service Installer][8]

1. On the next page:
    1. In the **Identity Application ID** box, enter the GUID for the app you're using for identity (from Azure AD).
    1. In the **Identity Application certificate file** box, enter (or browse to) the location of the certificate file.
    1. In the **Identity Application certificate password** box, enter the password for the certificate. This password is the one that you made note of when you used the script to create the certificates.
    1. In the **Azure Resource Manager root certificate file** box, enter (or browse to) the location of the certificate file.
    1. Select **Next**.

    ![Enter app ID and certificate info in App Service Installer][10]

1. For each of the three certificate file boxes, select **Browse**, and then go to the appropriate certificate file. You must provide the password for each certificate. These certificates are the ones that you created in the [Create required certificates step](azure-stack-app-service-before-you-get-started.md#get-certificates). Select **Next** after entering all the info.

    | Box | Certificate file name example |
    | --- | --- |
    | **App Service default SSL certificate file** | \_.appservice.local.AzureStack.external.pfx |
    | **App Service API SSL certificate file** | api.appservice.local.AzureStack.external.pfx |
    | **App Service Publisher SSL certificate file** | ftp.appservice.local.AzureStack.external.pfx |

    If you used a different domain suffix when you created the certificates, your certificate file names don't use *local.AzureStack.external*. Instead, use your custom domain info.

    ![Enter SSL certificate info in App Service Installer][11]

1. Enter the SQL Server details for the server instance used to host the App Service resource provider databases, and then select **Next**. The installer validates the SQL connection properties. You **must** enter either the internal IP or the FQDN for the SQL Server name.

    > [!NOTE]
    > The installer tries to test connectivity to the computer running SQL Server  before proceeding. However, if you chose to deploy into an existing virtual network, the installer might not be able to connect to the computer running SQL Server and displays a warning asking whether you want to continue. Verify the SQL Server info and continue if it's correct.
    >
    > From Azure App Service on Azure Stack 1.3 onward, the installer checks that the computer running SQL Server has database containment enabled at the SQL Server level. If it doesn't, you're prompted with the following exception:
    > ```sql
    >    Enable contained database authentication for SQL server by running below command on SQL server (Ctrl+C to copy)
    >    ***********************************************************
    >    sp_configure 'contained database authentication', 1;
    >    GO
    >    RECONFIGURE;
    >    GO
    >    ***********************************************************
    > ```
    > For more details, see the [release notes for Azure App Service on Azure Stack 1.3](azure-stack-app-service-release-notes-update-three.md).

    ![Enter SQL Server info in App Service Installer][12]

1. Review the role instance and SKU options. The defaults populate with the minimum number of instances and the minimum SKU for each role in an ASDK deployment. A summary of vCPU and memory requirements is provided to help plan your deployment. After you make your selections, select **Next**.

     > [!NOTE]
     > For production deployments, follow the guidance in [Capacity planning for Azure App Service server roles in Azure Stack](azure-stack-app-service-capacity-planning.md).
     >
     >

    | Role | Minimum instances | Minimum SKU | Notes |
    | --- | --- | --- | --- |
    | Controller | 1 | Standard_A2 - (2 vCPU, 3584 MB) | Manages and maintains the health of the App Service cloud. |
    | Management | 1 | Standard_A2 - (2 vCPUs, 3584 MB) | Manages the App Service Azure Resource Manager and API endpoints, portal extensions (admin, tenant, Functions portal), and the data service. To support failover, increase the recommended instances to 2. |
    | Publisher | 1 | Standard_A1 - (1 vCPU, 1792 MB) | Publishes content via FTP and web deployment. |
    | FrontEnd | 1 | Standard_A1 - (1 vCPU, 1792 MB) | Routes requests to App Service apps. |
    | Shared Worker | 1 | Standard_A1 - (1 vCPU, 1792 MB) | Hosts web or API apps and Azure Functions apps. You might want to add more instances. As an operator, you can define your offering and choose any SKU tier. The tiers must have a minimum of one vCPU. |

    ![Set role tiers and SKU options in App Service Installer][14]

    > [!NOTE]
    > Windows Server 2016 Core is *not* a supported platform image for use with Azure App Service on Azure Stack.  Don't use evaluation images for production deployments. Azure App Service on Azure Stack requires that Microsoft .NET 3.5.1 SP1 be activated on the image used for deployment. Marketplace-syndicated Windows Server 2016 images don't have this feature enabled. Therefore, you must create and use a Windows Server 2016 image with this feature pre-enabled.

1. In the **Select Platform Image** box, choose your deployment Windows Server 2016 virtual machine (VM) image from the images available on the compute resource provider for the App Service cloud. Select **Next**.

1. On the next page:
     1. Enter the Worker Role VM admin user name and password.
     2. Enter the Other Roles VM admin  user name and password.
     3. Select **Next**.

    ![Enter role VM admins in App Service Installer][16]

1. On the summary page:
    1. Verify the selections you made. To make changes, use the **Previous** buttons to visit previous pages.
    2. If the configurations are correct, select the check box.
    3. To start the deployment, select **Next**.

    ![Summary of selections made in App Service Installer][17]

1. On the next page:
    1. Track the installation progress. App Service on Azure Stack takes about 60 minutes to deploy, based on the default selections.
    2. After the installer finishes running, select **Exit**.

    ![Track installation process in App Service Installer][18]

## Post-deployment steps

> [!IMPORTANT]
> If you've provided the App Service RP with a SQL Always On Instance, you *must* [add the appservice_hosting and appservice_metering databases to an availability group](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/availability-group-add-a-database). You must also synchronize the databases to prevent any loss of service in the event of a database failover.

If you chose to deploy into an existing virtual network and an internal IP address to connect to your file server, you must add an outbound security rule, enabling SMB traffic between the worker subnet and the file server. In the administrator portal, go to the WorkersNsg Network Security Group and add an outbound security rule with the following properties:

- Source: Any
- Source port range: *
- Destination: IP addresses
- Destination IP address range: Range of IPs for your file server
- Destination port range: 445
- Protocol: TCP
- Action: Allow
- Priority: 700
- Name: Outbound_Allow_SMB445

## Validate the App Service on Azure Stack installation

1. In the Azure Stack administrator portal, go to **Administration - App Service**.

1. In the overview, under status, check to see that the **Status** displays **All roles are ready**.

    ![Overview in App Service Administration](media/azure-stack-app-service-deploy/image12.png)

## Test drive App Service on Azure Stack

After you deploy and register the App Service resource provider, test it to make sure that users can deploy web and API apps.

> [!NOTE]
> You must create an offer that has the Microsoft.Web namespace within the plan. Then, you need to have a tenant subscription that subscribes to this offer. For more info, see [Create offer](azure-stack-create-offer.md) and [Create plan](azure-stack-create-plan.md).
>
> You *must* have a tenant subscription to create apps that use App Service on Azure Stack. The only capabilities that a service admin can complete within the administrator portal are related to the resource provider administration of App Service. These capabilities include adding capacity, configuring deployment sources, and adding Worker tiers and SKUs.
>
> As of the third technical preview, to create web, API, and Azure Functions apps, you must use the tenant portal and have a tenant subscription.

1. In the Azure Stack tenant portal, select **+ Create a resource** > **Web + Mobile** > **Web App**.

1. On the **Web App** blade, type a name in the **Web app** box.

1. Under **Resource Group**, select **New**. Type a name in the **Resource Group** box.

1. Select **App Service plan/Location** > **Create New**.

1. On the **App Service plan** blade, type a name in the **App Service plan** box.

1. Select **Pricing tier** > **Free-Shared** or **Shared-Shared** > **Select** > **OK** > **Create**.

1. In less than a minute, a tile for the new web app appears on the dashboard. Select the tile.

1. On the **Web App** blade, select **Browse** to view the default website for this app.

## Deploy a WordPress, DNN, or Django website (optional)

1. In the Azure Stack tenant portal, select **+**, go to Azure Marketplace, deploy a Django website, and wait for successful completion. The Django web platform uses a file system-based database. It doesn't require any additional resource providers, such as SQL or MySQL.

1. If you also deployed a MySQL resource provider, you can deploy a WordPress website from Azure Marketplace. When you're prompted for database parameters, enter the user name as *User1\@Server1*, with the user name and server name of your choice.

1. If you also deployed a SQL Server resource provider, you can deploy a DNN website from Azure Marketplace. When you're prompted for database parameters, choose a database on the computer running SQL Server that's connected to your resource provider.

## Next steps

Prepare for additional admin operations for App Service on Azure Stack:

- [Capacity planning](azure-stack-app-service-capacity-planning.md)
- [Configure deployment sources](azure-stack-app-service-configure-deployment-sources.md)

<!--Image references-->
[1]: ./media/azure-stack-app-service-deploy-offline/app-service-exe-advanced-create-package.png
[2]: ./media/azure-stack-app-service-deploy-offline/app-service-exe-advanced-complete-offline.png
[3]: ./media/azure-stack-app-service-deploy-offline/app-service-azure-stack-arm-endpoints.png
[4]: ./media/azure-stack-app-service-deploy-offline/app-service-azure-stack-subscription-information.png
[5]: ./media/azure-stack-app-service-deploy-offline/app-service-default-VNET-config.png
[6]: ./media/azure-stack-app-service-deploy-offline/app-service-custom-VNET-config.png
[7]: ./media/azure-stack-app-service-deploy-offline/app-service-custom-VNET-config-with-values.png
[8]: ./media/azure-stack-app-service-deploy-offline/app-service-fileshare-configuration.png
[9]: ./media/azure-stack-app-service-deploy-offline/app-service-fileshare-configuration-error.png
[10]: ./media/azure-stack-app-service-deploy-offline/app-service-identity-app.png
[11]: ./media/azure-stack-app-service-deploy-offline/app-service-certificates.png
[12]: ./media/azure-stack-app-service-deploy-offline/app-service-sql-configuration.png
[13]: ./media/azure-stack-app-service-deploy-offline/app-service-sql-configuration-error.png
[14]: ./media/azure-stack-app-service-deploy-offline/app-service-cloud-quantities.png
[15]: ./media/azure-stack-app-service-deploy-offline/app-service-windows-image-selection.png
[16]: ./media/azure-stack-app-service-deploy-offline/app-service-role-credentials.png
[17]: ./media/azure-stack-app-service-deploy-offline/app-service-azure-stack-deployment-summary.png
[18]: ./media/azure-stack-app-service-deploy-offline/app-service-deployment-progress.png
