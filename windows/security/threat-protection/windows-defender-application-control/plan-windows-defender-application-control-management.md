---
title: Plan for WDAC policy management (Windows)
description: Learn about the decisions you need to make to establish the processes for managing and maintaining Windows Defender Application Control policies.
keywords: security, malware
ms.assetid: 8d6e0474-c475-411b-b095-1c61adb2bdbb
ms.prod: m365-security
ms.mktglfcycl: deploy
ms.sitesec: library
ms.pagetype: security
ms.localizationpriority: medium
audience: ITPro
ms.collection: M365-security-compliance
author: jsuther1974
ms.reviewer: isbrahm
ms.author: dansimp
manager: dansimp
ms.date: 02/21/2018
ms.technology: windows-sec
---

# Plan for Windows Defender Application Control lifecycle policy management

**Applies to:**

-   Windows 10
-   Windows 11
-   Windows Server 2016 and above

>[!NOTE]
>Some capabilities of Windows Defender Application Control (WDAC) are only available on specific Windows versions. Learn more about the [Windows Defender Application Control feature availability](feature-availability.md).

This topic describes the decisions you need to make to establish the processes for managing and maintaining Windows Defender Application Control (WDAC) policies.

## Policy XML lifecycle management

The first step in implementing application control is to consider how your policies will be managed and maintained over time. Developing a process for managing Windows Defender Application Control policies helps ensure that WDAC continues to effectively control how applications are allowed to run in your organization.

Most Windows Defender Application Control policies will evolve over time and proceed through a set of identifiable phases during their lifetime. Typically, these phases include:

1. [Define (or refine) the "circle-of-trust"](understand-windows-defender-application-control-policy-design-decisions.md) for the policy and build an audit mode version of the policy XML. In audit mode, block events are generated but files are not prevented from executing.
2. Deploy the audit mode policy to intended devices.
3. Monitor audit block events from the intended devices and add/edit/delete rules as needed to address unexpected/unwanted blocks.
4. Repeat steps 2-3 until the remaining block events meet expectations.
5. Generate the enforced mode version of the policy. In enforced mode, files that are not allowed by the policy are prevented from executing and corresponding block events are generated.
6. Deploy the enforced mode policy to intended devices. We recommend using staged rollouts for enforced policies to detect and respond to issues before deploying the policy broadly.
7. Repeat steps 1-6 anytime the desired "circle-of-trust" changes.  

![Recommended WDAC policy deployment process.](images/policyflow.png)

### Keep WDAC policies in a source control or document management solution

To effectively manage Windows Defender Application Control policies, you should store and maintain your policy XML documents in a central repository that is accessible to everyone responsible for WDAC policy management. We recommend a source control solution such as [GitHub](https://github.com/) or a document management solution such as [Office 365 SharePoint](https://products.office.com/sharepoint/collaboration), which provide version control and allow you to specify metadata about the XML documents.

### Set PolicyName, PolicyID, and Version metadata for each policy

Use the [Set-CIPolicyIDInfo](/powershell/module/configci/set-cipolicyidinfo) cmdlet to give each policy a descriptive name and set a unique ID in order to differentiate each policy when reviewing Windows Defender Application Control events or when viewing the policy XML document. Although you can specify a string value for PolicyId, for policies using the multiple policy format we recommend using the -ResetPolicyId switch to let the system autogenerate a unique ID for the policy.

> [!NOTE]
> PolicyID only applies to policies using the [multiple policy format](deploy-multiple-windows-defender-application-control-policies.md) on computers running Windows 10, version 1903 and above, or Windows 11. Running -ResetPolicyId on a policy created for pre-1903 computers will convert it to multiple policy format and prevent it from running on those earlier versions of Windows 10.
> PolicyID should be set only once per policy and use different PolicyID's for the audit and enforced mode versions of each policy.

In addition, we recommend using the [Set-CIPolicyVersion](/powershell/module/configci/set-cipolicyversion) cmdlet to increment the policy's internal version number when you make changes to the policy. The version must be defined as a standard four-part version string (e.g. "1.0.0.0").

### Policy rule updates

As new apps are deployed or existing apps are updated by the software publisher, you may need to make revisions to your rules to ensure that these apps run correctly. Whether policy rule updates are required will depend significantly on the types of rules your policy includes. Rules based on codesigning certificates provide the most resiliency against app changes while rules based on file attributes or hash are most likely to require updates when apps change. Alternatively, if you leverage WDAC [managed installer](configure-authorized-apps-deployed-with-a-managed-installer.md) functionality and consistently deploy all apps and their updates through your managed installer, then you are less likely to need policy updates.

## WDAC event management

Each time that a process is blocked by Windows Defender Application Control, events will be written to either the CodeIntegrity\Operational or the AppLocker\MSI and Script Windows event logs. The event details which file tried to run, the attributes of that file and its signatures, and the process that attempted to run the blocked file.

Collecting these events in a central location can help you maintain your Windows Defender Application Control policy and troubleshoot rule configuration problems. Event collection technologies such as those available in Windows allow administrators to subscribe to specific event channels and have the events from source computers aggregated into a forwarded event log on a Windows Server operating system collector. For more info about setting up an event subscription, see [Configure Computers to Collect and Forward Events](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc748890(v=ws.11)).

Additionally, Windows Defender Application Control events are collected by [Microsoft Defender for Endpoint](/microsoft-365/security/defender-endpoint/microsoft-defender-endpoint) and can be queried using the [advanced hunting](querying-application-control-events-centrally-using-advanced-hunting.md) feature.

## Application and user support policy

Considerations include:

- What type of end-user support is provided for blocked applications?
- How are new rules added to the policy?
- How are existing rules updated?
- Are events forwarded for review?

### Help desk support

If your organization has an established help desk support department in place, consider the following when deploying Windows Defender Application Control policies:

- What documentation does your support department require for new policy deployments?
- What are the critical processes in each business group both in work flow and timing that will be affected by application control policies and how could they affect your support department's workload?
- Who are the contacts in the support department?
- How will the support department resolve application control issues between the end user and those who maintain the Windows Defender Application Control rules?

### End-user support

Because Windows Defender Application Control is preventing unapproved apps from running, it is important that your organization carefully plan how to provide end-user support. Considerations include:

- Do you want to use an intranet site as a first line of support for users who have tried to run a blocked app?
- How do you want to support exceptions to the policy? Will you allow users to run a script to temporarily allow access to a blocked app?

## Document your plan

After deciding how your organization will manage your Windows Defender Application Control policy, record your findings.

- **End-user support policy.** Document the process that you will use for handling calls from users who have attempted to run a blocked app, and ensure that support personnel have clear escalation steps so that the administrator can update the Windows Defender Application Control policy, if necessary.
- **Event processing.** Document whether events will be collected in a central location called a store, how that store will be archived, and whether the events will be processed for analysis.
- **Policy management.** Detail what policies are planned, how they will be managed, and how rules will be maintained over time.
