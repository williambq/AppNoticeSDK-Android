#Upgrade App Notice SDK
*Current version: [2.0.0][version]*<br>
Last updated: April 22, 2016


##Upgrade from v.1.1.6 to 2.0.0
1. Replace the v.1.1.6 AppNoticeSDK.aar file in your project with the new 2.0.0 version.
2. Modify how the App Notice SDK is instantiated and started in your main activity. Change the call from:

    ```java
appNotice = new AppNotice(this, GHOSTERY_COMPANYID, GHOSTERY_NOTICEID, GHOSTERY_USEREMOTEVALUES, appNotice_callback);
appNotice.startConsentFlow();
    ```
To this (use the apporpriate start method for your consent flow type):

    ```java
appNotice = new AppNotice(this, GHOSTERY_COMPANYID, GHOSTERY_NOTICEID, appNotice_callback);
appNotice.startExplicitConsentFlow(); // To start an explicit consent flow
//appNotice.startImpliedConsentFlow(); // To start an implied consent flow
    ```

3. (Optional) The "ghostery_consent_flow_type" resource value has been deprecated and can safely be removed from your resource files. This functionality is now accomplished by the specific start method that is called. See step 2 above.
4. (Optional) The "ghostery_implied_flow_30day_display_max" has a new option. Setting this value to zero ("0") will now cause the App Notice SDK to only show the implied-consent flow on initial use (or after an SDK reset) and then again whenever the notice ID is changed.
