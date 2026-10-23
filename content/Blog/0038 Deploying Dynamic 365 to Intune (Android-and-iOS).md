---
title: "Deploying Dynamic 365 to Intune (Android & iOS)"
created: 2026-03-30
modified: 2026-03-30
tags: ["INTUNE"]
draft: false
---

## Introduction

The C-Suite Management made the decission to migrate from Salesforce over to Microsoft Dynamics 365 Sales to reduce license costs and unify the organization around Microsoft products. The sales team wanted to get Microsoft Dynamics 365 Sales on Android and iOS devices and I decided to assist them with onboarding the application since I'm currently the most experienced with Microsoft Intune.

In this article I'll be going through onboarding Microsoft Dynamics 365 Sales in Microsoft Intune.

## Android Deployment

This section of the article goes through setting up the application profile for Android endpoints.

1. Go to **Apps -> Android**.
    
    ![[0038 Deploying-Dynamic-365-to-Intune-(Android-and-iOS)-01.png]]

2. Click on **Create** 
    
    ![[0038 Deploying-Dynamic-365-to-Intune-(Android-and-iOS)-02.png]]

3. Select **Managed Google Play Store App**

    ![[0038 Deploying-Dynamic-365-to-Intune-(Android-and-iOS)-03.png]]

4. Find **Dynamic 365 Sales** on Enterprise Store and click on **Select**.

    ![[0038 Deploying-Dynamic-365-to-Intune-(Android-and-iOS)-04.png]]

5. Wait 5 minutes... And the app should show up on **Android Apps** section.

    ![[0038 Deploying-Dynamic-365-to-Intune-(Android-and-iOS)-05.png]]

6. Go to **Dynamic 365 Sales -> Properties** and configure who should have access to the app.

    ![[0038 Deploying-Dynamic-365-to-Intune-(Android-and-iOS)-06.png]]

## iOS Deployment

This section of the article goes through setting up application profile for iOS endpoints.

1. Go to **Apps -> iOS/iPadOS**

    ![[0038 Deploying-Dynamic-365-to-Intune-(Android-and-iOS)-07.png]]

2. Click on **Create**

    ![[0038 Deploying-Dynamic-365-to-Intune-(Android-and-iOS)-08.png]]

3. Select **iOS Store App**

    ![[0038 Deploying-Dynamic-365-to-Intune-(Android-and-iOS)-09.png]]

4. Click on  **Search the App Store**

    ![[0038 Deploying-Dynamic-365-to-Intune-(Android-and-iOS)-10.png]]

5. Search up **Dynamics 365 Sales** and select the app

    ![[0038 Deploying-Dynamic-365-to-Intune-(Android-and-iOS)-11.png]]

6. Enter and review the fields **Name, Description, Publisher, Applicable device types, and other informations**

    ![[0038 Deploying-Dynamic-365-to-Intune-(Android-and-iOS)-12.png]]

7. Select your Scope Tag

    ![[0038 Deploying-Dynamic-365-to-Intune-(Android-and-iOS)-13.png]]

8. Select the users/devices that should have access to the app

    ![[0038 Deploying-Dynamic-365-to-Intune-(Android-and-iOS)-14.png]]

9. Review the configuration and click on **Create**

    ![[0038 Deploying-Dynamic-365-to-Intune-(Android-and-iOS)-15.png]]

## Synchronization

The Microsoft Dynamics 365 Sales application will be installed on all the users/endpoints that is inside of the security group `ACL-Dynamics365-Users` within 8 hour time-frame. I would recommend setting it up so it's available at Company Portal marketplace instead of requiring the application to be installed on their endpoints.

## Conclusion

The reason a application profile has to be deployed in order for Sales Team to access Micrsofot Dynamics 365 Sales is because our Microsoft Intune configuration requires all applications that are accessed with company account to be managed by us. It's fairly easy to deploy Microsoft Dynamics 365 Sales application profile as you can see.
