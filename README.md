# App Notice SDK for Android<br>Installation and Customization
*Current version: [v2.2.0][version]*  
Last updated: April 25, 2017


## Prerequisites
* A valid App Notice Token and the set of Tracker ID's from Evidon. (Contact your Evidon Customer Success Manager to create or manage your App Notice configuration and to get these IDs.)
* Minimum supported Android SDK version: 14
* Android Support Library: v7 appcompat library

## Definitions
In this documentation, these terms are defined as follows:
* __Tracker:__ 3rd party SDKs (analytics, ad networks, affiliate tools, etc.) and internal user tracking.

## Compliance
To be in compliance, your app must honor a user's prior consent and withdrawl of consent related to tracking. The associated [Triangle app](https://github.com/evidon/AppNotice_Triangle_android_aar) demonstrates one way to do this for two different trackers. The sample code in this ReadMe document is from this [Triangle app](https://github.com/evidon/AppNotice_Triangle_android_aar).
* __Prior Consent:__ You must get a user's consent before any trackers are started.
* __Withdrawl of Consent:__ You must do one of these two things: 
  1. If a tracker is enabled and can be disabled or stopped in the current session, that tracker must be turned off in a way that it is no longer tracking the user in this session and future sessions.
  2. If a tracker is enabled and it can NOT be turned off or disabled in the current session, you must notify the user that they will continue to be tracked until the app is restarted. Then when the app is restarted, don't start the specified trackers.


## Use the App Notice SDK as a JCenter Dependency
This section covers how to implement the App Notice SDK into an Android Studio project using an AAR artifact from JCenter.

1. Make sure your project build.gradle file has JCenter listed as a repository as shown here:

  ```
  allprojects {
      repositories {
          jcenter()
      }
  }
  ```

2. Modify your module build.gradle file to add a dependency for the AppNoticeSDK.aar. Add a dependency for AppNoticeSDK.aar as shown here where "x.y.z" is the current SDK library version:

  ```
  dependencies {
      //...
      compile 'com.ghostery.privacy.appnoticesdk:AppNoticeSDK:x.y.z'
  }
  ```

3. Integrate the App Notice SDK into your code:

   3.1. Identify the appropriate location for starting the App Notice consent process. This is usually in the onCreate method of your main/start-up activity and should be before starting any user tracking or monitoring.

   3.2. The Android Studio SDK should automatically add these includes for you when the SDK code is added to your project. But if you need to add them manually, add these includes in the include section of the activity selected in step 3.1 above.

   ```java
   import com.evidon.privacy.appnoticesdk.AppNotice;
   import com.evidon.privacy.appnoticesdk.callbacks.AppNotice_Callback;
   ```

   3.3. Define these class variables in the activity selected in step 3.1 above (**Note: Use your own IDs and values**):

   ```java
   // Evidon variables
   // Note: Use your custom values for the Company ID, Notice ID and all or your tracker IDs. These test values won't work in your environment.
   private static final String EVIDON_TOKEN = "93cf713eb6cf45348563183d6d9d7184"; // My Evidon App Notice token (NOTE: Use your value here)

   // Evidon tracker IDs (NOTE: you will need to define a variable for each tracker you have in your app)
   private static final int EVIDON_TRACKERID_ADMOB = 464; // Tracker ID: AdMob
   private static final int EVIDON_TRACKERID_CRASHLYTICS = 3140; // Tracker ID: Crashlytics

   private static AppNotice appNotice; // Evidon App Notice SDK object
   private AppNotice_Callback appNotice_callback; // Evidon App Notice callback handler
   ```

   3.4. To start the App Notice process, you will need to create an App Notice call-back handler, instantiate the AppNotice object, and then start the App Notice flow as shown in the following example code:

   ```java
   @Override
   protected void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       setContentView(R.layout.activity_main);
       Context context = App.getContext();
       activity = this;
   
       AppCompatTextView sdkVersionTextView = (AppCompatTextView)findViewById(R.id.sdk_version);
       sdkVersionTextView.setText("SDK v." + AppNotice.sdkVersionName + "." + String.valueOf(AppNotice.sdkVersionCode));
   
       // Get the AdMob banner view and set an ad-loaded listner
       adView = (AdView) findViewById(R.id.adView);
       adView.setAdListener(new AdListener() {
           @Override
           public void onAdLoaded() {
   		// Handle ad-loaded event
           }
       });
   
       // Create the callback handler for the App Notice SDK
       appNotice_callback = new AppNotice_Callback() {
   
           // Called by the SDK when the user accepts or declines tracking from one of the consent notice screens
           @Override
           public void onOptionSelected(boolean isAccepted, HashMap<Integer, Boolean> appNotice_privacyPreferences) {
               // Handle your response
               if (isAccepted) {
                   manageTrackers(appNotice_privacyPreferences);
               } else {
                   // Toast invalid response state
                   Toast.makeText(activity, R.string.decline_state_error, Toast.LENGTH_LONG).show();
               }
           }
   
           // Called by the SDK when the startConsentFlow method is called and the SDK state indicates that the SDK doesn't need to be displayed.
           // The SDK is displayed in the following conditions and skipped in all others:
           //   - The Implied Consent screen:
           //     1) Has already been displayed the number of times specified by the parameter to the SDK's startConsentFlow method.
           //        0: Displays on first start and every notice ID change (recommended).
           //        1+: Is the max number of times to display the consent screen on start up in a 30-day period.
           //     2) Has already been displayed evidon_implied_flow_session_display_max times in the current session.
           //   - The Explicit Consent screen:
           //     1) Consent has already been given.
           @Override
           public void onNoticeSkipped(boolean isAccepted, HashMap<Integer, Boolean> trackerHashMap) {
               manageTrackers(trackerHashMap);
           }
   
           // Called by the SDK when the app-user is finished managing their privacy preferences on the Manage Preferences screen and navigates back to your app
           @Override
           public void onTrackerStateChanged(HashMap<Integer, Boolean> trackerHashMap) {
               manageTrackers(trackerHashMap);
           }
   
       };
   
       boolean isImplied = AppData.getString(AppData.APPDATA_CONSENT_FLOW_MODE, modeImplied).equals(modeImplied);
       if (isImplied) {
           // Example of instantiating the App Notice SDK in implied mode.
           // To be in compliance with honoring a user's prior consent, you must start this consent flow
           // before any trackers are started. In this demo, all trackers are only started from within
           // the manageTrackers method, and the manageTrackers method is only called from the App Notice
           // call-back handler. This ensures that trackers are only started with a users prior consent.
           appNotice = new AppNotice(this, EVIDON_TOKEN, appNotice_callback);
   
           // Start the implied consent flow (recommended)
           //   0: Displays on first start and every notice ID change (recommended).
           //   1+: Is the max number of times to display the consent screen on start up in a 30-day period.
           appNotice.startConsentFlow(IMPLIED_DEFAULT_BEHAVIOR);  // IMPLIED_DEFAULT_BEHAVIOR = 0
       } else {
           // Example of instantiating the App Notice SDK in explicit mode.
           // To be in compliance with honoring a user's prior consent, you must start this consent flow
           // before any trackers are started. In this demo, all trackers are only started from within
           // the manageTrackers method, and the manageTrackers method is only called from the App Notice
           // call-back handler. This ensures that trackers are only started with a users prior consent.
           appNotice = new AppNotice(this, EVIDON_TOKEN, appNotice_callback, IS_IMPLIED_MODE);  // IS_IMPLIED_MODE = false
   
           // Start the explicit consent flow:
           appNotice.startConsentFlow();
       }
   }
   ```

   3.5. The sample code above for the onCreate method calls the manageTrackers method three times in various ways depending on the user-selected state of the AdMob tracker. Here is an example of how the management of two trackers can be handled. Note that the AdMob tracker can be enabled and disabled in a single session, but for this demo, the Crashlytics tracker cannot be disabled once it is running and needs an app restart.

   ```java
   private void manageTrackers(HashMap<Integer, Boolean> trackerHashMap) {
        appRestartRequired = false;    // Assume the app doesn't need to be restarted to manage opt-outs

        if (trackerHashMap.size() > 0) {
            // == Manage AdMob ======================================
            // This demonstrates how to manage a tracker that can both be enabled and disabled in a
            // single session. The AdMob tracker is turned on and off as directed by a user's
            // privacy preferences.
            Boolean adMobEnabled = trackerHashMap.get(EVIDON_TRACKERID_ADMOB) == null? false : trackerHashMap.get(EVIDON_TRACKERID_ADMOB);

            if (adMobEnabled) {
                boolean inEmulator = Build.BRAND.toLowerCase().startsWith("generic");

                // Start the AdMob tracker as specified by the user
                // (Note: If there were a way to detect that this tracker were already running, we
                // could avoid restarting the tracker in that case.)
                AdRequest.Builder adRequestBuilder = new AdRequest.Builder();
                if (isTestingAds) {
                    if (inEmulator) {
                        adRequestBuilder.addTestDevice(AdRequest.DEVICE_ID_EMULATOR);
                    } else {
                        adRequestBuilder.addTestDevice("8E86E615D3646127F9A6DE11B6E8C533");
                    }
                }
                AdRequest adRequest = adRequestBuilder.build();

                adView.setVisibility(View.VISIBLE);
                adView.bringToFront();
                adView.loadAd(adRequest);

                // Toast the AdMob showing message (optional)
                Toast.makeText(this, TOAST_ADMOB_ENABLE, Toast.LENGTH_LONG).show();
            } else {
                // Stop the AdMob tracker as specified by the user:
                // To honor a user's withdrawn consent, if a tracker can be turned off or disabled,
                // that tracker must be turned off in a way that it is no longer tracking the user
                //  in this session and future sessions.
                adView.pause();
                adView.setVisibility(View.GONE);

                // Toast the AdMob disabled message (optional)
                Toast.makeText(this, TOAST_ADMOB_DISABLE, Toast.LENGTH_LONG).show();
            }

            // == Manage Crashlytics ================================
            // This demonstrates how to manage a tracker that can enabled but not disabled in a
            // single session. The Crashlytics tracker is turned on as directed by a user's
            // privacy preferences. But when a user requests that this tracker be turned off in the
            // privacy preferences, this demonstrates one way to notify that user to restart
            // the app.
            Boolean crashlyticsEnabled = trackerHashMap.get(EVIDON_TRACKERID_CRASHLYTICS) == null? false : trackerHashMap.get(EVIDON_TRACKERID_CRASHLYTICS);
            if (Fabric.isInitialized()) {    // Crashlytics is running in this session
                if (crashlyticsEnabled) {
                    // Toast the Crashlytics is enabled message (optional)
                    Toast.makeText(this, TOAST_CRASHLYTICS_ENABLE, Toast.LENGTH_LONG).show();
                } else {
                    // Remember to notify the user that an app restart is required to disable this tracker:
                    // To honor a user's withdrawn consent, if a tracker can NOT be turned off or
                    // disabled in the current session, you must notify the user that they will
                    // continue to be tracked until the app is restarted. Then when the app is
                    // restarted, don't start that tracker.
                    appRestartRequired = true;
                }
            } else { // Crashlytics has never been started in this session
                if (crashlyticsEnabled) {
                    // Start the Crashlytics tracker as specified by the user
                    Fabric.with(this, new Crashlytics());

                    // Toast the Crashlytics is enabled message (optional)
                    Toast.makeText(this, TOAST_CRASHLYTICS_ENABLE, Toast.LENGTH_LONG).show();
                } else {
                    // Do nothing: Crashlytics is disabled and not running

                    // Toast the Crashlytics is disabled message (optional)
                    Toast.makeText(this, TOAST_CRASHLYTICS_DISABLE, Toast.LENGTH_LONG).show();
                }
            }

        } else {
            Toast.makeText(activity, TOAST_TEXT_NOPREFS, Toast.LENGTH_LONG).show();
        }
    }
   ```

   3.6. The AppNotice_Callback handler must override these three methods as shown in section 3.4 above:

      *  **onOptionSelected**: This method is called by the SDK when the user accepts or declines tracking from either the Implied Consent dialog or the Explicit Consent dialog. This method has these two parameters:
        *  boolean isAccepted: True if the user clicked Accept on the Explicit Consent dialog or when they close the Implied Consent dialog. False if the user clicked Decline on the Explicit Consent dialog (the false state is deprecated and will be removed in a future version).
        *  HashMap<Integer, Boolean> trackerHashMap: A key/value map of all defined non-essential trackers. The key is the tracker ID and the value is true if the tracker is on and false if the tracker is off. **Note:** If the user's device is offline when the App Notice SDK first starts, the returned hashmap object will be empty. This state can be treated as if all optional trackers are on.
      *  **onNoticeSkipped**: This method is called by the SDK when the startConsentFlow method is called and the SDK state indicates that the SDK doesn't need to be displayed. The SDK is displayed in the following conditions and skipped in all others:
        * The Implied Consent dialog:
            1. Has already been displayed the number of times specified by the parameter to the SDK's startImpliedConsentFlow method.
                * 0: Displays on first start and every notice ID change (recommended).
                * 1+: Is the max number of times to display the consent screen on start up in a 30-day period.
            2. Has already been displayed evidon_implied_flow_session_display_max times in the current session.
        * The Explicit Consent dialog:
            1. Consent has already been given;
      * **onTrackerStateChanged**: This method is called by the SDK when the app-user is finished managing their privacy preferences on the Manage Preferences screen and navigates back your app. This method has this parameter:
        * HashMap<Integer, Boolean> trackerHashMap: A key/value map of all defined non-essential trackers. The key is the tracker ID and the value is true if the tracker is on and false if the tracker is off.

   3.7. In your callback methods, add code to handle responses as needed.
      * In the case where App Notice process returns true (accepted), you should handle the tracker information returned in the trackerPreferences map. Only enable/start tracking for trackers that are enabled, and disable/don’t start tracking for trackers that are disabled.
      * **Note:** In the case where the app-user declines the explicit notice, they are blocked from entering the app, so the App Notice process never returns false (declined).
      * In the case where onTrackerStateChanged is called and the app has already started trackers that are not turned off, either turn them off at this point, or inform the user that the applicable trackers will be disabled when the app is next started.
      * Notice that the provided sample code above, the code to initialize AdMob, has been moved into a new manageTrackers method to facilitate the various ways it can be managed. It also includes an example of how to turn this tracker off.

   3.8. The AppNotice constructor takes these parameters:
      * *FragmentActivity* **activity**: This is your activity from which this method is being called, usually your main/start-up activity. It will usually be “this” or “this.getActivity”. This can also be subclasses of FragmentActivity, like AppCompatActivity or ActionBarActivity.
      * *String* **appNoticeToken**: The notice token for the configuration created for this app.
      * *AppNotice_Callback* **appNotice_callback**: Your app will need to instantiate an AppNotice_Callback object from the SDK and override it's callback methods as described above.
      * *boolean* **isImpliedMode**: Initialize the SDK in either implied or explicit mode: true = implied; false = explicit.

4. You can start the Manage Privacy Preferences activity directly from a menu or settings screen in your app by calling this method from a button or menu click handler (Note: This assumes appNotice has been initialized as shown earlier):

   ```java
   appNotice.showManagePreferences();
   ```

   4.1. Since your app is already running at this time, you may not be able to disable or stop all trackers that the user has disabled. In this case, you **must** notify your user that the tracker changes will be applied the next time the app starts up.

5. After the end user has completed one of the SDK consent flows, you can get the current tracker preferences on start up by calling this method (Note: This assumes other code and variables are defined as shown in earlier sample code):

  ```java
  manageTrackers(appNotice.getTrackerPreferences());
  ```

6. As mentioned earlier, the SDK will skip displaying the implied and explicit dialogs depending on acceptance status, session count and 30-day count. To reset these acceptance and count values in the SDK so that you can force the dialogs to be displayed, you can reset the SDK by calling the following method (Note: This assumes appNotice has been initialized as shown earlier):

  ```java
  appNotice.resetSDK();
  ```

7. (Optional) The App Notice SDK makes both the SDK version name and the version code available to the host app via these two static variables:

  ```java
  String AppNotice.sdkVersionName
  int AppNotice.sdkVersionCode
  ```


## Declined Consent Best Practices
To be in privacy notification compliance, when the SDK calls back to your app and indicates that the user has declined consent, your app can only proceed if all user tracking is disabled. These are options for how to best handle this case:
* If your app does not require any trackers to provide full functionality, then the app can proceed by disabling all trackers and letting the user continue using the app.
* If the app does have required trackers that cannot be disabled, then the app must prevent the user from proceeding to use the app, or at least the parts of the app that require trackers.
If your app will allow the user to continue to use the app with limited functionality, notify the user about the limitations. You could display a dialog with text similar to this: "To enjoy the full functionality of this app, you must either upgrade to the paid version or accept the privacy preferences in this version. Please restart this app if you would like to accept. This app will now continue with limited functionality."


## Support Multiple App Versions
* To support versions of your app that each have a different set of trackers, use unique App Notice configurations (with unique tokens) in each version of your app.
* Ask your Evidon Customer Success Manager to create an App Notice configuration for each version of your app that has a different combination of trackers.
* After creating an App Notice, be sure to use that App Notice's token in the applicable version of your app when you instantiate the App Notice SDK inside your app. For example, when you instantiate the App Notice object, use the new value for the token in this method call:

```java
appNotice = new AppNotice(this, EVIDON_TOKEN, appNotice_callback, IS_IMPLIED_MODE);
```


## SDK Customization
You can customize text and color in the AAR-based App Notice SDK for Android. You do this by overriding the SDK's resource values in your app with parameters of the same name.

### Text Customization:
To change the message on the Consent screen, add the applicable string parameter to your app's string resource file and customize the text:

  ```xml
  <!-- Text of the message nn the Implied Consent screen. -->
  <string name="evidon_manage_preferences_implied_message">This app uses technologies so that we, and our partners, can remember you and understand how you use our app. To see a list of these technologies and choose whether they can be used, please manage your preferences below. Further use of this app will be considered consent.</string>
  <!-- Text of the message on the Explicit Consent screen. -->
  <string name="evidon_manage_preferences_explicit_message">This app uses technologies so that we, and our partners, can remember you and understand how you use our app. To see a list of these technologies and choose whether they can be used, please manage your preferences.\n\nBefore proceeding, you must accept, decline or manage your privacy preferences below.</string>
  ```

### Theme Customization:
To change the theme of the App Notice SDK between light (default) and dark, add one of the following style parameter to your app's style resource file:

  ```xml
  <!-- Light Theme (default) -->
  <style name="evidon_AppNoticeTheme" parent="evidon_AppNoticeTheme" />
  ```

  ```xml
  <!-- Dark Theme -->
  <style name="evidon_AppNoticeTheme" parent="evidon_AppNoticeTheme.Base.Dark" />
  ```

### Color Customization:
To change the color of various elements in the App Notice SDK using Material Design colors, add one or more of the following color parameters to your app's color resource file and customize the colors as desired:

  ```xml
  <!-- Evidon SDK colors - light theme -->
  <color name="evidonColorPrimary">#03a9f4</color>
  <color name="evidonColorPrimaryDark">#0288D1</color>
  <color name="evidonColorAccent">#40c4ff</color>

  <!-- Evidon SDK colors - dark theme -->
  <color name="evidonColorPrimary.Dark">#000000</color>
  <color name="evidonColorPrimaryDark.Dark">#0288D1</color>
  <color name="evidonColorAccent.Dark">#40c4ff</color>

  <!-- Evidon SDK misc custom colors -->
  <color name="evidonColorButtonPrimary">#03A9F4</color>
  <color name="evidonColorButtonTextPrimary">#FFFFFF</color>
  <color name="evidonColorButtonSecondary">#CCCCCC</color>
  <color name="evidonColorButtonTextSecondary">#666666</color>
  <color name="evidonColorButtonTextBorderless">#03A9F4</color>
  <color name="evidonColorPrivacyTools">#9FA2A4</color>
  ```

### Advanced Customization:
Almost all elements in the App Notice SDK UI are customizable by overiding elements in the SDK's resource files. To get full access to all of the SDK resource files, contact your Evidon Customer Support Manager (CSM). Evidon does not support problems caused by this level of customization...proceed at your own risk.

### SDK Configuration Parameters:
These configuration parameters may be used to modify how the App Notice SDK operates. If you include any of these parameters in your config.xml file, your specified value will override the default value specified in the SDK. The values shown below are the default values.

  ```xml
  <!-- Configuration-->
  <!-- If true, the web-based tab will be shown, else it will not be shown -->
  <bool name="evidon_show_web_tab">false</bool>

  <!-- Max times to display the Implied consent flow in an app session. -->
  <integer name="evidon_implied_flow_session_display_max">1</integer>

  <!-- System -->
  <!-- HTTP connect and read timeouts, in millis -->
  <integer name="evidon_http_connect_timeout">15000</integer>
  <integer name="evidon_http_read_timeout">10000</integer>
  ```


## Troubleshooting
* If your app has a mismatch between the Android target SDK (targetSdkVersion) and the Android build libraries (buildToolsVersion), and you are using ProGuard in your app, you may see a compiler warning about not being able to find the referenced method 'android.content.res.ColorStateList getColorStateList'. In this case, you will need to add this line to your ProGuard configuration:

  ```xml
  -dontwarn android.content.res.**
  ```
