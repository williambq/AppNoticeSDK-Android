App Notice SDK Installation & Customization Instructions
========================================================

*Current version: [v1.0][version]*
August 2015


Android Studio Using AAR
------------------------

1.	Copy the InAppConsentSDK.aar file from the AppNotice_aar.zip package into your module's libs folder.
2.	Add the following configuration information to your module's AndroidManifest.xml file:
  1. Inside the application section and below your activities, add these two Ghostery activities to the right:
  
```
<activity
    android:name="com.ghostery.privacy.inappconsentsdk.app.TrackerListActivity"
    android:launchMode="singleInstance"
    android:configChanges="orientation|keyboardHidden|screenSize" android:screenOrientation="unspecified" android:alwaysRetainTaskState="true" android:clearTaskOnLaunch="false"
    >
</activity>
<activity
    android:name="com.ghostery.privacy.inappconsentsdk.app.TrackerDetailActivity"
    android:launchMode="singleInstance"
    android:configChanges="orientation|keyboardHidden|screenSize" android:screenOrientation="unspecified" android:alwaysRetainTaskState="true" android:clearTaskOnLaunch="false"
    >
</activity>
```
3\.	Modify your module’s build.gradle file to add a dependency for the InAppConsentSDK.aar:
  1.	Add flatDir section to repositories section as shown here:
  ```
  repositories {
    mavenCentral()
    flatDir {
        dirs 'libs' // Allow the .aar file to be found in the libs folder
    }
}
```
  2.	Add a Library dependency for InAppConsentSDK.aar as shown here:
  ```
  dependencies {
    …
    compile 'com.ghostery.privacy.inappconsentsdk:InAppConsentSDK:@aar'
}
```

4\.	Integrate the App Notice SDK into your code:
  1.	Identify the appropriate location for starting the In-App Consent process. This is usually in the onCreate method of your main/start-up activity and should be before starting any user tracking or monitoring.
  2.	The Android Studio SDK should automatically add these includes for you when the SDK code is added to your project. But if you need to add them manually, add these includes in the include section of the activity selected in step 4.1 above.
  ```
  import com.ghostery.privacy.inappconsentsdk.callbacks.InAppConsent_Callback;
\import com.ghostery.privacy.inappconsentsdk.model.InAppConsent;
```

  3.	Define these class variables in the activity selected in step 4.1 above (**Note: Use your own IDs and values**):
  ```
  // Ghostery variables
private static final int GHOSTERY_COMPANYID = 242; // My Ghostery company ID
private static final int GHOSTERY_NOTICEID = 6107; // The Ghostery notice ID for this app
private static final int GHOSTERY_TRACKERID_ADMOB = 464; // Tracker ID: AdMob (note: you will need to define a variable for each tracker you have in your app)
private static final boolean GHOSTERY_USEREMOTEVALUES = true; // If true, causes SDK to override local SDK settings with those defined in the Ghostery Admin Portal
private InAppConsent inAppConsent; // Ghostery App Notice SDK object
private InAppConsent_Callback inAppConsent_callback; // Ghostery In-App Consent callback handler
private HashMap<Integer, Boolean> inAppConsent_privacyPreferences; // Map of non-essential trackers (by ID) and their on/off states
```

  4.	To start the In-App Consent process, you will need to create an In-App Consent call-back handler, instantiate the InAppConsent object, and then start the In-App Consent flow as shown in the following example code:
  
  ```
  @Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    // Create the callback handler for the App Notice SDK
    inAppConsent_callback = new InAppConsent_Callback() {

        // Called by the SDK when the user accepts or declines tracking from one of the Consent flow dialogs
        @Override
        public void onOptionSelected(boolean isAccepted, HashMap<Integer, Boolean> trackerHashMap) {
            // Handle your response
            if (isAccepted) {
                inAppConsent_privacyPreferences = trackerHashMap;
                initAdMob(isAccepted && inAppConsent_privacyPreferences.get(GHOSTERY_TRACKERID_ADMOB));
            } else {
                // Note: The decline-case only needs to be handled for Explicit notice types
                try {
                    DeclineConfirmation_DialogFragment dialog = new DeclineConfirmation_DialogFragment();
                    dialog.show(getFragmentManager(), "DeclineConfirmation_DialogFragment");
                } catch (IllegalStateException e) {
                    Log.e(TAG, "Error while trying to display the decline-confirmation dialog.", e);
                }
            }
        }

        // Called by the SDK when startConsentFlow is called but the SDK state meets one or more of the following conditions:
        //   - The Implied Consent dialog has been already been displayed ghostery_ric_session_max_default times in the current session.
        //   - The Implied Consent dialog has already been displayed ghostery_ric_max_default times in the last 30 days.
        //   - The Explicit Consent dialog has already been accepted.
        @Override
        public void onNoticeSkipped() {
            inAppConsent_privacyPreferences = inAppConsent.getTrackerPreferences();
            initAdMob(inAppConsent_privacyPreferences.get(GHOSTERY_TRACKERID_ADMOB));
        }

        // Called by the SDK when the app-user is finished managing their privacy preferences on the Manage Preferences screen and navigates back your app
        @Override
        public void onTrackerStateChanged(HashMap<Integer, Boolean> trackerHashMap) {
            inAppConsent_privacyPreferences = trackerHashMap;
            initAdMob(inAppConsent_privacyPreferences.get(GHOSTERY_TRACKERID_ADMOB));
        }
    };

    // Instantiate and start the Ghostery In-App Consent flow
    inAppConsent = new InAppConsent(this);
    inAppConsent.startConsentFlow(GHOSTERY_COMPANYID, GHOSTERY_NOTICEID, GHOSTERY_USEREMOTEVALUES, inAppConsent_callback);
}
```

  1.	The sample code above for the onCreate method calls the initAdMob method three times in various way depending on the user-selected state of the AdMob tracker. Here is an example of how the initialization of the AdMob 3rd-party object can be handled:

```
private void initAdMob(boolean isOn) {
    // Load an ad into the AdMob banner view.
    AdView adView = (AdView) findViewById(R.id.adView);

    if (isOn) {
        AdRequest adRequest = new AdRequest.Builder().build();
        adView.setVisibility(View.VISIBLE);
        adView.loadAd(adRequest);

        // Toasts the AdMob showing message
        Toast.makeText(this, TOAST_TEXT_SHOW, Toast.LENGTH_LONG).show();
    } else {
        adView.pause();
        adView.setVisibility(View.GONE);

        // Toasts the AdMob disabled message
        Toast.makeText(this, TOAST_TEXT_DISABLE, Toast.LENGTH_LONG).show();
    }
}
```

  2. The InAppConsent_Callback handler must override these three methods as shown above:

    1.  **onOptionSelected**: This method is called by the SDK when the user accepts or declines tracking from either the Implied Consent dialog or the Explicit Consent dialog. This method has these two parameters:
      1.  boolean isAccepted: True if the user clicked Accept on the Explicit Consent dialog or when they close the Implied Consent dialog. False if the user clicked Decline on the Explicit Consent dialog.
      2.  HashMap<Integer, Boolean> trackerHashMap: A key/value map of all defined non-essential trackers. The key is the tracker ID and the value is true if the tracker is on and false if the tracker is off.
    2.  **onNoticeSkipped**: This method is called by the SDK when inAppConsent.startConsentFlow is called and the SDK determines that a notice dialog does not need to be displayed. This condition can happen when the notice has been displayed the maximum specified times in a session (ghostery_ric_session_max_default), the maximum times in a 30-day period (ghostery_ric_max_default); or if the Explicit Consent notice was accepted previously. The max parameters can be edited in the SDK resources or passed in from the service.
    3. **onTrackerStateChanged**: This method is called by the SDK when the app-user is finished managing their privacy preferences on the Manage Preferences screen and navigates back your app. This method has this parameter:
•	HashMap<Integer, Boolean> trackerHashMap: A key/value map of all defined non-essential trackers. The key is the tracker ID and the value is true if the tracker is on and false if the tracker is off.

4.	In your callback methods, add code to handle responses as needed.
  1. In the case where In-App Consent process returns true (accepted), you should handle the tracker information returned in the trackerPreferences map. Only enable/start tracking for trackers that are enabled, and disable/don’t start tracking for trackers that are disabled.
  2. In the case where the explicit In-App Consent process returns false (declined), you should notify the user about the issue, and then lock or close your app before processing any customer tracking (see example code).
  3. In the case where onTrackerStateChanged is called and the app has already started trackers that are not turned off, either turn them off at this point, or inform the user that the applicable trackers will be disabled when the app is next started.
  4. Notice that the provided sample code above, the code to initialize AdMob has been moved into a new initAdMob method to facilitate the various ways it can be managed. It also included an example of how to turn this tracker off.

4.	The startConsentFlow method takes these parameters:
  1. FragmentActivity activity: This is your activity from which this method is being called, usually your main/start-up activity. It will usually be “this” or “this.getActivity”. This can also be subclasses of FragmentActivity, like AppCompatActivity or ActionBarActivity.
  2. int company_id: The company ID assigned to you by Ghostery.
  3. int pub_notice_id: The Pub-notice ID of the configuration created for this app.
  4. boolean useRemoteValues: True = Try to use the In-App Consent dialog configuration parameters from the service. If the parameters can't be retrieved or are missing, the parameters defined in the resource files will be used. False = Use local resource values where possible. In this case, the only information used from the JSON data is the “bric” parameter to determine if the flow is for Implied Consent or Explicit Consent, and the Tracker data array.
5.	You can start the Manage Privacy Preferences activity directly from a menu or settings screen in your app by calling this method from a button or menu click handler (Note: This assumes inAppConsent has been initialized as shown earlier):
```
inAppConsent.showManagePreferences(GHOSTERY_COMPANYID, GHOSTERY_NOTICEID, GHOSTERY_USEREMOTEVALUES, inAppConsent_callback);
```

  1.	Since your app is already running at this time, you may not be able to disable or stop all trackers that the user has disabled. In this case, you **must** notify your user that the tracker changes will be applied the next time the app starts up.

6\.	After the end user has completed one of the SDK consent flows, you can get the current tracker preferences on start up by calling this method (Note: This assumes other code and variables are defined as shown in earlier sample code):
```
inAppConsent_privacyPreferences = inAppConsent.getTrackerPreferences();
initAdMob(inAppConsent_privacyPreferences.get(GHOSTERY_TRACKERID_ADMOB));
```

7\.	As mentioned earlier, the SDK will skip displaying the implied and explicit dialogs depending on acceptance status, session count and 30-day count. To reset these acceptance and count values in the SDK so that you can force the dialogs to be displayed, you can reset the SDK by calling the following method (Note: This assumes inAppConsent has been initialized as shown earlier):

8\.	Customization via resource files (**optional**):
  1.	Unzip the InAppConsentSDK.aar file.
  2.	Edit the strings and colors in ```…\res\values\values.xml```
  3.	Edit layouts in ```…\src\main\res\layout\*.xml```

| | |
| :-----------------------|----------------------------------------------|
| Tracker Detail activity | ghostery_activity_tracker_detail.xml         |
| Tracker List activity   | ghostery_activity_tracker_list.xml           |
| Tracker List activity   | ghostery_explicitinfo_dialogfragment.xml     |
|Explicit Consent flow dialog | ghostery_explicitinfo_dialogfragment.xml |
|Tracker Learn More fragment  | ghostery_fragment_learn_more.xml         |
| Tracker Detail fragment     | ghostery_fragment_tracker_detail.xml     |
| Implied Consent flow dialog | ghostery_impliedinfo_dialogfragment.xml  |
| Tracker list item           | ghostery_tracker_list_item.xml           |

  4\.	Re-zip the modified files back into InAppConsentSDK.aar and use this modified AAR file in your module.


[version]: https://github.com/ghostery/AppNoticeSDK-Android
