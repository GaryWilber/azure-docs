---
title: 'Tutorial: Azure Active Directory single sign-on (SSO) integration with EAB Navigate Strategic Care | Microsoft Docs'
description: Learn how to configure single sign-on between Azure Active Directory and EAB Navigate Strategic Care.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/20/2019
ms.author: jeedes
---

# Tutorial: Azure Active Directory single sign-on (SSO) integration with EAB Navigate Strategic Care

In this tutorial, you'll learn how to integrate EAB Navigate Strategic Care with Azure Active Directory (Azure AD). When you integrate EAB Navigate Strategic Care with Azure AD, you can:

* Control in Azure AD who has access to EAB Navigate Strategic Care.
* Enable your users to be automatically signed-in to EAB Navigate Strategic Care with their Azure AD accounts.
* Manage your accounts in one central location - the Azure portal.

To learn more about SaaS app integration with Azure AD, see [What is application access and single sign-on with Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## Prerequisites

To get started, you need the following items:

* An Azure AD subscription. If you don't have a subscription, you can get a [free account](https://azure.microsoft.com/free/).
* EAB Navigate Strategic Care single sign-on (SSO) enabled subscription.

## Scenario description

In this tutorial, you configure and test Azure AD SSO in a test environment.

* EAB Navigate Strategic Care supports **SP** initiated SSO

## Adding EAB Navigate Strategic Care from the gallery

To configure the integration of EAB Navigate Strategic Care into Azure AD, you need to add EAB Navigate Strategic Care from the gallery to your list of managed SaaS apps.

1. Sign in to the [Azure portal](https://portal.azure.com) using either a work or school account, or a personal Microsoft account.
1. On the left navigation pane, select the **Azure Active Directory** service.
1. Navigate to **Enterprise Applications** and then select **All Applications**.
1. To add new application, select **New application**.
1. In the **Add from the gallery** section, type **EAB Navigate Strategic Care** in the search box.
1. Select **EAB Navigate Strategic Care** from results panel and then add the app. Wait a few seconds while the app is added to your tenant.


## Configure and test Azure AD single sign-on for EAB Navigate Strategic Care

Configure and test Azure AD SSO with EAB Navigate Strategic Care using a test user called **B.Simon**. For SSO to work, you need to establish a link relationship between an Azure AD user and the related user in EAB Navigate Strategic Care.

To configure and test Azure AD SSO with EAB Navigate Strategic Care, complete the following building blocks:

1. **[Configure Azure AD SSO](#configure-azure-ad-sso)** - to enable your users to use this feature.
    1. **[Create an Azure AD test user](#create-an-azure-ad-test-user)** - to test Azure AD single sign-on with B.Simon.
    1. **[Assign the Azure AD test user](#assign-the-azure-ad-test-user)** - to enable B.Simon to use Azure AD single sign-on.
1. **[Configure EAB Navigate Strategic Care SSO](#configure-eab-navigate-strategic-care-sso)** - to configure the single sign-on settings on application side.
    1. **[Create EAB Navigate Strategic Care test user](#create-eab-navigate-strategic-care-test-user)** - to have a counterpart of B.Simon in EAB Navigate Strategic Care that is linked to the Azure AD representation of user.
1. **[Test SSO](#test-sso)** - to verify whether the configuration works.

## Configure Azure AD SSO

Follow these steps to enable Azure AD SSO in the Azure portal.

1. In the [Azure portal](https://portal.azure.com/), on the **EAB Navigate Strategic Care** application integration page, find the **Manage** section and select **single sign-on**.
1. On the **Select a single sign-on method** page, select **SAML**.
1. On the **Set up single sign-on with SAML** page, click the edit/pen icon for **Basic SAML Configuration** to edit the settings.

   ![Edit Basic SAML Configuration](common/edit-urls.png)

1. On the **Basic SAML Configuration** section, enter the values for the following fields:

    In the **Sign-on URL** text box, type a URL using the following pattern:
    `https://<CUSTOMERURL>.eab.com`

	> [!NOTE]
	> The value is not real. Update the value with the actual Sign-On URL. Contact [EAB Navigate Strategic Care Client support team](mailto:tech@gradesfirst.com) to get the value. You can also refer to the patterns shown in the **Basic SAML Configuration** section in the Azure portal.

1. On the **Set up single sign-on with SAML** page, In the **SAML Signing Certificate** section, click the copy button to copy **App Federation Metadata Url** and save it on your computer.

	![The Certificate download link](common/copy-metadataurl.png)

### Create an Azure AD test user

In this section, you'll create a test user in the Azure portal called B.Simon.

1. From the left pane in the Azure portal, select **Azure Active Directory**, select **Users**, and then select **All users**.
1. Select **New user** at the top of the screen.
1. In the **User** properties, follow these steps:
   1. In the **Name** field, enter `B.Simon`.  
   1. In the **User name** field, enter the username@companydomain.extension. For example, `B.Simon@contoso.com`.
   1. Select the **Show password** check box, and then write down the value that's displayed in the **Password** box.
   1. Click **Create**.

### Assign the Azure AD test user

In this section, you'll enable B.Simon to use Azure single sign-on by granting access to EAB Navigate Strategic Care.

1. In the Azure portal, select **Enterprise Applications**, and then select **All applications**.
1. In the applications list, select **EAB Navigate Strategic Care**.
1. In the app's overview page, find the **Manage** section and select **Users and groups**.

   ![The "Users and groups" link](common/users-groups-blade.png)

1. Select **Add user**, then select **Users and groups** in the **Add Assignment** dialog.

	![The Add User link](common/add-assign-user.png)

1. In the **Users and groups** dialog, select **B.Simon** from the Users list, then click the **Select** button at the bottom of the screen.
1. If you're expecting any role value in the SAML assertion, in the **Select Role** dialog, select the appropriate role for the user from the list and then click the **Select** button at the bottom of the screen.
1. In the **Add Assignment** dialog, click the **Assign** button.

## Configure EAB Navigate Strategic Care SSO

To configure single sign-on on **EAB Navigate Strategic Care** side, you need to send the **App Federation Metadata Url** to [EAB Navigate Strategic Care support team](mailto:tech@gradesfirst.com). They set this setting to have the SAML SSO connection set properly on both sides.

### Create EAB Navigate Strategic Care test user

In this section, you create a user called B.Simon in EAB Navigate Strategic Care. Work with [EAB Navigate Strategic Care support team](mailto:tech@gradesfirst.com) to add the users in the EAB Navigate Strategic Care platform. Users must be created and activated before you use single sign-on.

## Test SSO 

In this section, you test your Azure AD single sign-on configuration using the Access Panel.

When you click the EAB Navigate Strategic Care tile in the Access Panel, you should be automatically signed in to the EAB Navigate Strategic Care for which you set up SSO. For more information about the Access Panel, see [Introduction to the Access Panel](https://support.microsoft.com/account-billing/sign-in-and-start-apps-from-the-my-apps-portal-2f3b1bae-0e5a-4a86-a33e-876fbd2a4510).

## Additional resources

- [List of Tutorials on How to Integrate SaaS Apps with Azure Active Directory](./tutorial-list.md)

- [What is application access and single sign-on with Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [What is conditional access in Azure Active Directory?](../conditional-access/overview.md)

- [Try EAB Navigate Strategic Care with Azure AD](https://aad.portal.azure.com/)