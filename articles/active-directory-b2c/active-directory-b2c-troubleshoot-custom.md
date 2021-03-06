---
title: 'Azure Active Directory B2C: Troubleshoot Custom Policies | Microsoft Docs'
description: A topic on how to get troubleshoot problems with Azure Active Directory B2C custom policies
services: active-directory-b2c
documentationcenter: ''
author: saeeda
manager: krassk
editor: parakhj

ms.assetid: 658c597e-3787-465e-b377-26aebc94e46d
ms.service: active-directory-b2c
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: article
ms.devlang: na
ms.date: 04/04/2017
ms.author: saeeda
---

# Azure Active Directory B2C: Collecting Logs

This article provides steps for collecting logs from Azure AD B2C so that you can diagnose problems with your custom policies.

## Use Application Insights

Azure AD B2C supports a feature for sending data to Application Insights.  Application Insights provides a way to diagnose exceptions and visualize application performance issues.

### Setup Application Insights

1. Go to the [Azure portal](https://portal.azure.com). Ensure you are in the tenant with your Azure subscription (not your Azure AD B2C tenant).
1. Click **+ New** in the left-hand navigation menu.
1. Search for and select **Application Insights**, then click **Create**.
1. Complete the form and click **Create**. Select **General** for the **Application Type**.
1. Once the resource has been created, open the Application Insights resource.
1. Find **Properties** in the left-menu, and click on it.
1. Copy the **Instrumentation Key** and save it for the next section.

### Set up the custom policy

1. Open the RP file (e.g. SignUpOrSignin.xml).
1. Add the following attributes to the `<TrustFrameworkPolicy>` element:

  ```XML
  UserJourneyRecorderEndpoint="urn:journeyrecorder:applicationinsights"
  ```

1. If it doesn't exist already, add a child node `<UserJourneyBehaviors>` to the `<RelyingParty>` node.
2. Add the following node as a child of the `<UserJourneyBehaviors>` element. Make sure to replace `{Your Application Insights Key}` with the **Instrumentation Key** that you obtained in the previous section.

  ```XML
  <JourneyInsights TelemetryEngine="ApplicationInsights" InstrumentationKey="{Your Application Insights Key}" DeveloperMode="true" ClientEnabled="false" ServerEnabled="true" TelemetryVersion="1.0.0" />
  ```

  * `DeveloperMode="true"` tells ApplicationInsights to expedite the telemetry through the processing pipeline, good for development, but constrained at high volumes.
  * `ClientEnabled="true"` sends the ApplicationInsights client-side script for tracking page view and client-side errors (not needed).
  * `ServerEnabled="true"` sends the existing UserJourneyRecorder JSON as a custom event to Application Insights.
  The final XML will look like the following:

  ```XML
  <TrustFrameworkPolicy
    ...
    TenantId="fabrikamb2c.onmicrosoft.com"
    PolicyId="SignUpOrSignInWithAAD"
    UserJourneyRecorderEndpoint="urn:journeyrecorder:applicationinsights"
  >
    ...
    <RelyingParty>
      <UserJourneyBehaviors>
        <JourneyInsights TelemetryEngine="ApplicationInsights" InstrumentationKey="{Your Application Insights Key}" DeveloperMode="true" ClientEnabled="false" ServerEnabled="true" TelemetryVersion="1.0.0" />
      </UserJourneyBehaviors>
      ...
  </TrustFrameworkPolicy>
  ```

3. Upload the policy.

### See the logs

>[!NOTE]
> There is a short delay (less than 5 minutes) before you can see new logs in Application Insights.

1. Open the Application Insights resource that you created in the [Azure portal](https://portal.azure.com).
1. In the **Overview** menu, click on **Analytics**.
1. Open a new tab in Application Insights.
1. Here is a list of queries you can use to see the logs

| Query | Description |
|---------------------|--------------------|
traces | See all of the logs generated by Azure AD B2C |
traces \| where timestamp > ago(1d) | See all of the logs generated by Azure AD B2C for the last day

You can learn more about the Analytics tool [here](https://docs.microsoft.com/azure/application-insights/app-insights-analytics).




