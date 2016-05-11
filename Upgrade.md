#Upgrade App Notice SDK
*Current version: [2.0.1][version]*<br>
Last updated: May 11, 2016


##Upgrade from v.1.1.6 to 2.0.1
1. Replace the v.1.1.6 AppNoticeSDK.aar file in your project with the new 2.0.1 version.
2. Modify how the App Notice SDK is instantiated and started in your main activity. Change the call from:

    ```java
appNotice = new AppNotice(this, GHOSTERY_COMPANYID, GHOSTERY_NOTICEID, GHOSTERY_USEREMOTEVALUES, appNotice_callback);
appNotice.startConsentFlow();
    ```
To this (use the apporpriate start method for your consent flow type):

    ```java
appNotice = new AppNotice(this, GHOSTERY_COMPANYID, GHOSTERY_NOTICEID, appNotice_callback);

// Start the implied-consent flow (recommended)
//   0 displays on first start and every notice ID change (recommended).
//   1+ is the max number of times to display the consent screen on start up in a 30-day period.
appNotice.startImpliedConsentFlow(0);

// (Alternate:)
// Start the explicit-consent flow in either strict or lenient mode:
//   true = use strict mode (end user must click Accept to continue).
//   false = use lenient mode (on decline, the consent flow screen is only displayed again when the notice ID changes).
//appNotice.startExplicitConsentFlow(true);
    ```

3. (Optional) The "ghostery_implied_flow_30day_display_max" resource value has been deprecated and can safely be removed from your resource files. This functionality is now accomplished by calling the SDK's startImpliedConsentFlow method with an integer value that specifies the number of times to display the implied consent screen on start up. See step 2 above. There is also a new option. Setting this value to zero ("0") will now cause the App Notice SDK to only show the implied-consent flow on initial use (or after an SDK reset) and then again whenever the notice ID is changed.
4. (Optional) The "ghostery_consent_flow_type" resource value has been deprecated and can safely be removed from your resource files. This functionality is now accomplished by calling the SDK's startExplicitConsentFlow method with a boolean value that specifies the specific start method that is to be used. See step 2 above.
