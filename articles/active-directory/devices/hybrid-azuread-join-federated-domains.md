---
title: Configure hybrid Azure Active Directory join for federated domains | Microsoft Docs
description: Learn how to configure hybrid Azure Active Directory join for federated domains.

services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: tutorial
ms.date: 05/20/2020

ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sandeo

#Customer intent: As an IT admin, I want to set up hybrid Azure Active Directory (Azure AD) joined devices for federated domains so I can automatically create and manage device identities in Azure AD for my Active Directory domain-joined computers

ms.collection: M365-identity-device-management
---
# Tutorial: Configure hybrid Azure Active Directory join for federated domains

Like a user in your organization, a device is a core identity you want to protect. You can use a device's identity to protect your resources at any time and from any location. You can accomplish this goal by bringing device identities and managing them in Azure Active Directory (Azure AD) by using one of the following methods:

- Azure AD join
- Hybrid Azure AD join
- Azure AD registration

Bringing your devices to Azure AD maximizes user productivity through single sign-on (SSO) across your cloud and on-premises resources. You can secure access to your cloud and on-premises resources with [Conditional Access](../conditional-access/howto-conditional-access-policy-compliant-device.md) at the same time.

A federated environment should have an identity provider that supports the following requirements. If you have a federated environment using Active Directory Federation Services (AD FS), then the below requirements are already supported.

- **WIAORMULTIAUTHN claim:** This claim is required to do hybrid Azure AD join for Windows down-level devices.
- **WS-Trust protocol:** This protocol is required to authenticate Windows current hybrid Azure AD joined devices with Azure AD.
  When you're using AD FS, you need to enable the following WS-Trust endpoints: 
   `/adfs/services/trust/2005/windowstransport`
   `/adfs/services/trust/13/windowstransport`
   `/adfs/services/trust/2005/usernamemixed`
   `/adfs/services/trust/13/usernamemixed`
   `/adfs/services/trust/2005/certificatemixed`
   `/adfs/services/trust/13/certificatemixed` 

> [!WARNING] 
> Both **adfs/services/trust/2005/windowstransport** and **adfs/services/trust/13/windowstransport** should be enabled as intranet facing endpoints only and must NOT be exposed as extranet facing endpoints through the Web Application Proxy. To learn more on how to disable WS-Trust Windows endpoints, see [Disable WS-Trust Windows endpoints on the proxy](/windows-server/identity/ad-fs/deployment/best-practices-securing-ad-fs#disable-ws-trust-windows-endpoints-on-the-proxy-ie-from-extranet). You can see what endpoints are enabled through the AD FS management console under **Service** > **Endpoints**.

In this tutorial, you learn how to configure hybrid Azure AD join for Active Directory domain-joined computers devices in a federated environment by using AD FS.

You learn how to:

> [!div class="checklist"]
> * Configure hybrid Azure AD join
> * Enable Windows downlevel devices
> * Verify the registration
> * Troubleshoot

## Prerequisites

This tutorial assumes that you're familiar with these articles:

- [What is a device identity?](overview.md)
- [How to plan your hybrid Azure AD join implementation](hybrid-azuread-join-plan.md)
- [How to do controlled validation of hybrid Azure AD join](hybrid-azuread-join-control.md)

To configure the scenario in this tutorial, you need:

- Windows Server 2012 R2 with AD FS
- [Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594) version 1.1.819.0 or later

Beginning with version 1.1.819.0, Azure AD Connect includes a wizard that you can use to configure hybrid Azure AD join. The wizard significantly simplifies the configuration process. The related wizard:

- Configures the service connection points (SCPs) for device registration
- Backs up your existing Azure AD relying party trust
- Updates the claim rules in your Azure AD trust

The configuration steps in this article are based on using the Azure AD Connect wizard. If you have an earlier version of Azure AD Connect installed, you must upgrade it to 1.1.819 or later to use the wizard. If installing the latest version of Azure AD Connect isn't an option for you, see [how to manually configure hybrid Azure AD join](hybrid-azuread-join-manual.md).

Hybrid Azure AD join requires devices to have access to the following Microsoft resources from inside your organization's network:  

- `https://enterpriseregistration.windows.net`
- `https://login.microsoftonline.com`
- `https://device.login.microsoftonline.com`
- Your organization's Security Token Service (STS) (For federated domains)
- `https://autologon.microsoftazuread-sso.com` (If you use or plan to use seamless SSO)

> [!WARNING]
> If your organization uses proxy servers that intercept SSL traffic for scenarios like data loss prevention or Azure AD tenant restrictions, ensure that traffic to 'https://device.login.microsoftonline.com' is excluded from TLS break-and-inspect. Failure to exclude 'https://device.login.microsoftonline.com' may cause interference with client certificate authentication, causing issues with device registration and device-based Conditional Access.

Beginning with Windows 10 1803, if the instantaneous hybrid Azure AD join for a federated environment by using AD FS fails, we rely on Azure AD Connect to sync the computer object in Azure AD that's subsequently used to complete the device registration for hybrid Azure AD join. Verify that Azure AD Connect has synced the computer objects of the devices you want to be hybrid Azure AD joined to Azure AD. If the computer objects belong to specific organizational units (OUs), you must also configure the OUs to sync in Azure AD Connect. To learn more about how to sync computer objects by using Azure AD Connect, see [Configure filtering by using Azure AD Connect](../hybrid/how-to-connect-sync-configure-filtering.md#organizational-unitbased-filtering).

> [!NOTE]
> To get device registration sync join to succeed, as part of the device registration configuration, do not exclude the default device attributes from your Azure AD Connect sync configuration. To learn more about default device attributes synced to AAD, see [Attributes synchronized by Azure AD Connect](../hybrid/reference-connect-sync-attributes-synchronized.md#windows-10).

If your organization requires access to the internet via an outbound proxy, Microsoft recommends [implementing Web Proxy Auto-Discovery (WPAD)](/previous-versions/tn-archive/cc995261(v%3dtechnet.10)) to enable Windows 10 computers for device registration with Azure AD. If you encounter issues configuring and managing WPAD, see [Troubleshoot automatic detection](/previous-versions/tn-archive/cc302643(v=technet.10)). 

If you don't use WPAD and want to configure proxy settings on your computer, you can do so beginning with Windows 10 1709. For more information, see [Configure WinHTTP settings by using a group policy object (GPO)](/archive/blogs/netgeeks/winhttp-proxy-settings-deployed-by-gpo).

> [!NOTE]
> If you configure proxy settings on your computer by using WinHTTP settings, any computers that can't connect to the configured proxy will fail to connect to the internet.

If your organization requires access to the internet via an authenticated outbound proxy, you must make sure that your Windows 10 computers can successfully authenticate to the outbound proxy. Because Windows 10 computers run device registration by using machine context, you must configure outbound proxy authentication by using machine context. Follow up with your outbound proxy provider on the configuration requirements.

To verify if the device is able to access the above Microsoft resources under the system account, you can use [Test Device Registration Connectivity](/samples/azure-samples/testdeviceregconnectivity/testdeviceregconnectivity/) script.

## Configure hybrid Azure AD join

To configure a hybrid Azure AD join by using Azure AD Connect, you need:

- The credentials of a global administrator for your Azure AD tenant  
- The enterprise administrator credentials for each of the forests
- The credentials of your AD FS administrator

**To configure a hybrid Azure AD join by using Azure AD Connect**:

1. Start Azure AD Connect, and then select **Configure**.

   ![Welcome](./media/hybrid-azuread-join-federated-domains/11.png)

1. On the **Additional tasks** page, select **Configure device options**, and then select **Next**.

   ![Additional tasks](./media/hybrid-azuread-join-federated-domains/12.png)

1. On the **Overview** page, select **Next**.

   ![Overview](./media/hybrid-azuread-join-federated-domains/13.png)

1. On the **Connect to Azure AD** page, enter the credentials of a global administrator for your Azure AD tenant, and then select **Next**.

   ![Connect to Azure AD](./media/hybrid-azuread-join-federated-domains/14.png)

1. On the **Device options** page, select **Configure Hybrid Azure AD join**, and then select **Next**.

   ![Device options](./media/hybrid-azuread-join-federated-domains/15.png)

1. On the **SCP** page, complete the following steps, and then select **Next**:

   ![SCP](./media/hybrid-azuread-join-federated-domains/16.png)

   1. Select the forest.
   1. Select the authentication service. You must select **AD FS server** unless your organization has exclusively Windows 10 clients and you have configured computer/device sync, or your organization uses seamless SSO.
   1. Select **Add** to enter the enterprise administrator credentials.

1. On the **Device operating systems** page, select the operating systems that the devices in your Active Directory environment use, and then select **Next**.

   ![Device operating system](./media/hybrid-azuread-join-federated-domains/17.png)

1. On the **Federation configuration** page, enter the credentials of your AD FS administrator, and then select **Next**.

   ![Federation configuration](./media/hybrid-azuread-join-federated-domains/18.png)

1. On the **Ready to configure** page, select **Configure**.

   ![Ready to configure](./media/hybrid-azuread-join-federated-domains/19.png)

1. On the **Configuration complete** page, select **Exit**.

   ![Configuration complete](./media/hybrid-azuread-join-federated-domains/20.png)

## Enable Windows downlevel devices

If some of your domain-joined devices are Windows downlevel devices, you must:

- Configure the local intranet settings for device registration
- Install Microsoft Workplace Join for Windows downlevel computers

> [!NOTE]
> Windows 7 support ended on January 14, 2020. For more information, [Support for Windows 7 has ended](https://support.microsoft.com/en-us/help/4057281/windows-7-support-ended-on-january-14-2020).

### Configure the local intranet settings for device registration

To successfully complete hybrid Azure AD join of your Windows downlevel devices and to avoid certificate prompts when devices authenticate to Azure AD, you can push a policy to your domain-joined devices to add the following URLs to the local intranet zone in Internet Explorer:

- `https://device.login.microsoftonline.com`
- Your organization's STS (For federated domains)
- `https://autologon.microsoftazuread-sso.com` (For seamless SSO)

You also must enable **Allow updates to status bar via script** in the user’s local intranet zone.

### Install Microsoft Workplace Join for Windows downlevel computers

To register Windows downlevel devices, organizations must install [Microsoft Workplace Join for non-Windows 10 computers](https://www.microsoft.com/download/details.aspx?id=53554). Microsoft Workplace Join for non-Windows 10 computers is available in the Microsoft Download Center.

You can deploy the package by using a software distribution system like [Microsoft Endpoint Configuration Manager](/configmgr/). The package supports the standard silent installation options with the `quiet` parameter. The current branch of Configuration Manager offers benefits over earlier versions, like the ability to track completed registrations.

The installer creates a scheduled task on the system that runs in the user context. The task is triggered when the user signs in to Windows. The task silently joins the device with Azure AD by using the user credentials after it authenticates with Azure AD.

## Verify the registration

Here are 3 ways to locate and verify the device state:

### Locally on the device

1. Open Windows PowerShell.
2. Enter `dsregcmd /status`.
3. Verify that both **AzureAdJoined** and **DomainJoined** are set to **YES**.
4. You can use the **DeviceId** and compare the status on the service using either the Azure portal or PowerShell.

For downlevel devices see the article [Troubleshooting hybrid Azure Active Directory joined down-level devices](troubleshoot-hybrid-join-windows-legacy.md#step-1-retrieve-the-registration-status)

### Using the Azure portal

1. Go to the devices page using a [direct link](https://portal.azure.com/#blade/Microsoft_AAD_IAM/DevicesMenuBlade/Devices).
2. Information on how to locate a device can be found in [How to manage device identities using the Azure portal](./device-management-azure-portal.md).
3. If the **Registered** column says **Pending**, then Hybrid Azure AD Join has not completed. In federated environments, this can happen only if it failed to register and AAD connect is configured to sync the devices.
4. If the **Registered** column contains a **date/time**, then Hybrid Azure AD Join has completed.

### Using PowerShell

Verify the device registration state in your Azure tenant by using **[Get-MsolDevice](/powershell/module/msonline/get-msoldevice)**. This cmdlet is in the [Azure Active Directory PowerShell module](/powershell/azure/active-directory/install-msonlinev1).

When you use the **Get-MSolDevice** cmdlet to check the service details:

- An object with the **device ID** that matches the ID on the Windows client must exist.
- The value for **DeviceTrustType** is **Domain Joined**. This setting is equivalent to the **Hybrid Azure AD joined** state on the **Devices** page in the Azure AD portal.
- For devices that are used in Conditional Access, the value for **Enabled** is **True** and **DeviceTrustLevel** is **Managed**.

1. Open Windows PowerShell as an administrator.
2. Enter `Connect-MsolService` to connect to your Azure tenant.

#### Count all Hybrid Azure AD joined devices (excluding **Pending** state)

```azurepowershell
(Get-MsolDevice -All -IncludeSystemManagedDevices | where {($_.DeviceTrustType -eq 'Domain Joined') -and (([string]($_.AlternativeSecurityIds)).StartsWith("X509:"))}).count
```

#### Count all Hybrid Azure AD joined devices with **Pending** state

```azurepowershell
(Get-MsolDevice -All -IncludeSystemManagedDevices | where {($_.DeviceTrustType -eq 'Domain Joined') -and (-not([string]($_.AlternativeSecurityIds)).StartsWith("X509:"))}).count
```

#### List all Hybrid Azure AD joined devices

```azurepowershell
Get-MsolDevice -All -IncludeSystemManagedDevices | where {($_.DeviceTrustType -eq 'Domain Joined') -and (([string]($_.AlternativeSecurityIds)).StartsWith("X509:"))}
```

#### List all Hybrid Azure AD joined devices with **Pending** state

```azurepowershell
Get-MsolDevice -All -IncludeSystemManagedDevices | where {($_.DeviceTrustType -eq 'Domain Joined') -and (-not([string]($_.AlternativeSecurityIds)).StartsWith("X509:"))}
```

#### List details of a single device:

1. Enter `get-msoldevice -deviceId <deviceId>` (This is the **DeviceId** obtained locally on the device).
2. Verify that **Enabled** is set to **True**.

## Troubleshoot your implementation

If you experience issues with completing hybrid Azure AD join for domain-joined Windows devices, see:

- [Troubleshooting devices using dsregcmd command](./troubleshoot-device-dsregcmd.md)
- [Troubleshoot hybrid Azure AD join for Windows current devices](troubleshoot-hybrid-join-windows-current.md)
- [Troubleshoot hybrid Azure AD join for Windows downlevel devices](troubleshoot-hybrid-join-windows-legacy.md)

## Next steps

Learn how to [manage device identities by using the Azure portal](device-management-azure-portal.md).

<!--Image references-->
[1]: ./media/active-directory-conditional-access-automatic-device-registration-setup/12.png
