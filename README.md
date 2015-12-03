#App Notice SDK Installation & Customization Instructions
*Current version: [v1.1.0][version]*
November 2015


##Prerequisites
*	Minimum supported Android SDK version: 15
*	Android Support Library: v7 appcompat library


##Compliance
To be in compliance, your app must honor a user's prior consent and withdrawl of consent related to tracking. The associated [Triangle app](https://github.com/ghostery/AppNotice_Triangle_android_aar) demonstrates one way to do this for two different trackers. The sample code in this ReadMe document is from this [Triangle app](https://github.com/ghostery/AppNotice_Triangle_android_aar).
* __Prior Consent:__ You must get a user's consent before any trackers are started. 
* __Withdrawl of Consent:__ You must do one of these two things: 
  1. If a tracker is enabled and can be disabled or stopped in the current session, that tracker must be turned off in a way that it is no longer tracking the user in this session and future sessions.
  2. If a tracker is enabled and it can NOT be turned off or disabled in the current session, you must notify the user that they will continue to be tracked until the app is restarted. Then when the app is restarted, don't start the specified trackers.


##Android Studio Using AAR
This section covers how to implement the App Notice SDK into an Android Studio project from an AAR file.

1. Copy the AppNoticeSDK.aar file from the AppNotice_aar.zip package into your module's libs folder.
2. In your module's AndroidManifest.xml file, inside the application section and below your activities, add these two Ghostery activities:

  ```xml
<!-- Include the Ghostery AddNotice activities -->
<activity
    android:name="com.ghostery.privacy.appnoticesdk.app.TrackerListActivity"
    android:launchMode="singleInstance"
    android:configChanges="orientation|keyboardHidden|screenSize" android:screenOrientation="unspecified" android:alwaysRetainTaskState="true" android:clearTaskOnLaunch="false"
    >
</activity>
<activity
    android:name="com.ghostery.privacy.appnoticesdk.app.TrackerDetailActivity"
    android:launchMode="singleInstance"
    android:configChanges="orientation|keyboardHidden|screenSize" android:screenOrientation="unspecified" android:alwaysRetainTaskState="true" android:clearTaskOnLaunch="false"
    >
</activity>
  ```

3. Modify your module’s build.gradle file to add a dependency for the AppNoticeSDK.aar:
  1. Add flatDir section to repositories section as shown here:

    ```
repositories {
  mavenCentral()
  flatDir {
      dirs 'libs' // Allow the .aar file to be found in the libs folder
  }
}
    ```

  2. Add a Library dependency for AppNoticeSDK.aar as shown here:

    ```
dependencies {
    //...
    compile 'com.ghostery.privacy.appnoticesdk:AppNoticeSDK:@aar'
}
    ```

4. Integrate the App Notice SDK into your code:
  1. Identify the appropriate location for starting the App Notice consent process. This is usually in the onCreate method of your main/start-up activity and should be before starting any user tracking or monitoring.
  2. The Android Studio SDK should automatically add these includes for you when the SDK code is added to your project. But if you need to add them manually, add these includes in the include section of the activity selected in step 4.1 above.

    ```java
import com.ghostery.privacy.appnoticesdk.AppNotice;
import com.ghostery.privacy.appnoticesdk.callbacks.AppNotice_Callback;
    ```

  3.	Define these class variables in the activity selected in step 4.1 above (**Note: Use your own IDs and values**):

    ```java
// Ghostery variables
// Note: Use your custom values for the Company ID, Notice ID and all or your tracker IDs. These test values won't work in your environment.
private static final int GHOSTERY_COMPANYID = 242; // My Ghostery company ID (NOTE: Use your value here)
//private static final int GHOSTERY_CONFIGID = 6690; // The Ghostery configuration ID for this app (NOTE: Use your value here) (Implied)
private static final int GHOSTERY_CONFIGID = 6691; // The Ghostery configuration ID for this app (NOTE: Use your value here) (Explicit)

// Ghostery tracker IDs (NOTE: you will need to define a variable for each tracker you have in your app)
private static final int GHOSTERY_TRACKERID_ADMOB = 464; // Tracker ID: AdMob
private static final int GHOSTERY_TRACKERID_CRASHLYTICS = 3140; // Tracker ID: Crashlytics

private static final boolean GHOSTERY_USEREMOTEVALUES = false; // If true, causes SDK to override local SDK settings with those defined in the Ghostery Admin Portal
private AppNotice appNotice; // Ghostery App Notice SDK object
private AppNotice_Callback appNotice_callback; // Ghostery App Notice callback handler
boolean appRestartRequired; // Ghostery parameter to track if app needs to be restarted after opt-out
    ```

  4.	To start the App Notice process, you will need to create an App Notice call-back handler, instantiate the AppNotice object, and then start the App Notice flow as shown in the following example code:
  
    ```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    activity = this;

    // Create the callback handler for the App Notice SDK
    appNotice_callback = new AppNotice_Callback() {

        // Called by the SDK when the user accepts or declines tracking from one of the consent flow dialogs
        @Override
        public void onOptionSelected(boolean isAccepted, HashMap<Integer, Boolean> appNotice_privacyPreferences) {
            // Handle your response
            if (isAccepted) {
                manageTrackers(appNotice_privacyPreferences);
            } else {
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
            manageTrackers(appNotice.getTrackerPreferences());
        }

        // Called by the SDK when the app-user is finished managing their privacy preferences on the Manage Preferences screen and navigates back your app
        @Override
        public void onTrackerStateChanged(HashMap<Integer, Boolean> trackerHashMap) {
            manageTrackers(trackerHashMap);
        }
    };

    // Instantiate and start the Ghostery consent flow:
    // To be in compliance with honoring a user's prior consent, you must start this consent flow
    // before any trackers are started. In this demo, all trackers are only started from within
    // the manageTrackers method, and the manageTrackers method is only called from the App Notice
    // call-back handler. This ensures that trackers are only started with a users prior consent.
    appNotice = new AppNotice(this, GHOSTERY_COMPANYID, GHOSTERY_CONFIGID, GHOSTERY_USEREMOTEVALUES, appNotice_callback);
    appNotice.startConsentFlow();
}
    ```

  5. The sample code above for the onCreate method calls the manageTrackers method three times in various way depending on the user-selected state of the AdMob tracker. Here is an example of how the management of two trackers can be handled. Note that the AdMob tracker can be enabled and disabled in a single session, but for this demo, the Crashlytics tracker cannot be disabled once it is running and needs extra handling.

    ```java
private void manageTrackers(HashMap<Integer, Boolean> trackerHashMap) {
appRestartRequired = false;	// Assume the app doesn't need to be restarted to manage opt-outs

    if (trackerHashMap.size() > 0) {
        // == Manage AdMob ======================================
	// This demonstrates how to manage a tracker that can both be enabled and disabled in a
	// single session. The AdMob tracker is turned on and off as directed by a user's
	// privacy preferences.
        Boolean adMobEnabled = trackerHashMap.get(GHOSTERY_TRACKERID_ADMOB) == null? false : trackerHashMap.get(GHOSTERY_TRACKERID_ADMOB);
        // Get the AdMob banner view
        AdView adView = (AdView) findViewById(R.id.adView);

        if (adMobEnabled) {
		// Start the AdMob tracker as specified by the user
            AdRequest adRequest = new AdRequest.Builder().build();
            adView.setVisibility(View.VISIBLE);
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
        Boolean crashlyticsEnabled = trackerHashMap.get(GHOSTERY_TRACKERID_CRASHLYTICS) == null? false : trackerHashMap.get(GHOSTERY_TRACKERID_CRASHLYTICS);
	if (Fabric.isInitialized()) {	// Crashlytics is running in this session
		if (crashlyticsEnabled) {
			// Do nothing: Crashlytics is enabled and running
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

  6. The AppNotice_Callback handler must override these three methods as shown above:

      *  **onOptionSelected**: This method is called by the SDK when the user accepts or declines tracking from either the Implied Consent dialog or the Explicit Consent dialog. This method has these two parameters:
        *  boolean isAccepted: True if the user clicked Accept on the Explicit Consent dialog or when they close the Implied Consent dialog. False if the user clicked Decline on the Explicit Consent dialog.
        *  HashMap<Integer, Boolean> trackerHashMap: A key/value map of all defined non-essential trackers. The key is the tracker ID and the value is true if the tracker is on and false if the tracker is off.
      *  **onNoticeSkipped**: This method is called by the SDK when appNotice.startConsentFlow is called and the SDK determines that a notice dialog does not need to be displayed. This condition can happen when the notice has been displayed the maximum specified times in a session (ghostery_ric_session_max_default), the maximum times in a 30-day period (ghostery_ric_max_default); or if the Explicit Consent notice was accepted previously. The max parameters can be edited in the SDK resources or passed in from the service.
      * **onTrackerStateChanged**: This method is called by the SDK when the app-user is finished managing their privacy preferences on the Manage Preferences screen and navigates back your app. This method has this parameter:
        *	HashMap<Integer, Boolean> trackerHashMap: A key/value map of all defined non-essential trackers. The key is the tracker ID and the value is true if the tracker is on and false if the tracker is off.

  7.	In your callback methods, add code to handle responses as needed.
      * In the case where App Notice process returns true (accepted), you should handle the tracker information returned in the trackerPreferences map. Only enable/start tracking for trackers that are enabled, and disable/don’t start tracking for trackers that are disabled.
      * In the case where the explicit App Notice process returns false (declined), you should notify the user about the issue, and then lock or close your app before processing any customer tracking (see example code).
      * In the case where onTrackerStateChanged is called and the app has already started trackers that are not turned off, either turn them off at this point, or inform the user that the applicable trackers will be disabled when the app is next started.
      * Notice that the provided sample code above, the code to initialize AdMob has been moved into a new manageTrackers method to facilitate the various ways it can be managed. It also included an example of how to turn this tracker off.

  8.	The startConsentFlow method takes these parameters:
      * FragmentActivity activity: This is your activity from which this method is being called, usually your main/start-up activity. It will usually be “this” or “this.getActivity”. This can also be subclasses of FragmentActivity, like AppCompatActivity or ActionBarActivity.
      * int company_id: The company ID assigned to you by Ghostery.
      * int pub_notice_id: The Pub-notice ID of the configuration created for this app.
      * boolean useRemoteValues: True = Try to use the App Notice consent dialog configuration parameters from the service. If the parameters can't be retrieved or are missing, the parameters defined in the resource files will be used. False = Use local resource values where possible. In this case, the only information used from the JSON data is the “bric” parameter to determine if the flow is for Implied Consent or Explicit Consent, and the Tracker data array.

5.	You can start the Manage Privacy Preferences activity directly from a menu or settings screen in your app by calling this method from a button or menu click handler (Note: This assumes appNotice has been initialized as shown earlier):

  ```java
appNotice.showManagePreferences();
  ```

  1.	Since your app is already running at this time, you may not be able to disable or stop all trackers that the user has disabled. In this case, you **must** notify your user that the tracker changes will be applied the next time the app starts up.

6.	After the end user has completed one of the SDK consent flows, you can get the current tracker preferences on start up by calling this method (Note: This assumes other code and variables are defined as shown in earlier sample code):

  ```java
manageTrackers(appNotice.getTrackerPreferences());
  ```

7.	As mentioned earlier, the SDK will skip displaying the implied and explicit dialogs depending on acceptance status, session count and 30-day count. To reset these acceptance and count values in the SDK so that you can force the dialogs to be displayed, you can reset the SDK by calling the following method (Note: This assumes appNotice has been initialized as shown earlier):

  ```java
appNotice.resetSDK();
  ```

##Support Multiple App Versions
* To support versions of your app that each have a different set of trackers, use unique App Notice configurations in each version of your app.
* You can use your Ghostery control panel website (https://my.ghosteryenterprise.com) to create an App Notice configuration for each version of your app that has a different combination of trackers.
* After creating an App Notice, be sure to use that App Notice's ID in the applicable version of your app when you interact with the App Notice SDK inside your app. For example, when you instantiate the App Notice consent object, use the new value for noticeId in this method call: 

  ```java
appNotice = new AppNotice(this, GHOSTERY_COMPANYID, GHOSTERY_NOTICEID, GHOSTERY_USEREMOTEVALUES, appNotice_callback);
  ```

[version]: https://github.com/ghostery/AppNoticeSDK-Android

##Customization Via Resource Files (optional)
1. Unzip AppNoticeSDK.aar to a new folder  outside of your project.
2. Edit applicable strings and values in the this file: ...\res\values\values.xml

    <table>
    <tr><th>Description</th><th><b>Resource String Name</b></th><th><b>Default Value</b></th></tr>
    <tr><th>From file:</th><td colspan="2">From: file:.../res/values/ghostery_strings.xml</td></tr>
    <tr><td>Default consent flow type: true = Explicit, false = Implied</td><td>ghostery_bric</td><td>true</td></tr>
    <tr><th>From file:</th><td colspan="2">.../res/values/ghostery_colors.xml</td></tr>
    <tr><td>Background color for both the Implied and the Explicit consent dialogs</td><td>ghostery_dialog_background_color</td><td>#E5E5E5</td></tr>
    <tr><td>Dialog button color for all but Explicit Decline</td><td>ghostery_dialog_button_color</td><td>#333333</td></tr>
    <tr><td>Dialog button text color for all but Explicit Decline</td><td>ghostery_dialog_explicit_accept_button_text_color</td><td>#FFFFFF</td></tr>
    <tr><td>Dialog button color for Explicit Decline</td><td>ghostery_dialog_explicit_decline_button_color</td><td>#333333</td></tr>
    <tr><td>Dialog button text color for Explicit Decline</td><td>ghostery_dialog_explicit_decline_button_text_color</td><td>#FFFFFF</td></tr>
    <tr><td>Header text color for both the Implied and the Explicit consent dialogs</td><td>ghostery_dialog_header_text_color</td><td>#000000</td></tr>
    <tr><td>Message text color for both the Implied and the Explicit consent dialogs</td><td>ghostery_dialog_message_text_color</td><td>#000000</td></tr>
    <tr><td>Section divider line for both the vendor list and the vendor detail screens</td><td>ghostery_divider_color</td><td>#46AAAAAA</td></tr>
    <tr><td>Action Bar/Header background color for both the vendor list and the vendor detail screens</td><td>ghostery_header_background_color</td><td>#009688</td></tr>
    <tr><td>Action Bar/Header text color for both the vendor list and the vendor detail screens</td><td>ghostery_header_text_color</td><td>#000000</td></tr>
    <tr><td>Vendor list and list item background color</td><td>ghostery_list_background</td><td>#FFFFFF</td></tr>
    <tr><td>Color of the divider used between categories in the vendor list</td><td>ghostery_list_category_divider_color</td><td>#AAAAAA</td></tr>
    <tr><td>Color of the Category header text in the vendor list</td><td>ghostery_list_category_text_color</td><td>#9FA2A4</td></tr>
    <tr><td>Color of the divider used between list items in the vendor list (default matches background color so it disapeares)</td><td>ghostery_list_divider_color</td><td>#FFFFFF</td></tr>
    <tr><td>Color of the vendor name text in the vendor list</td><td>ghostery_list_vendorname_text_color</td><td>#212121</td></tr>
    <tr><td>Color of the vendor message text at the top of the privacy preferences screen</td><td>ghostery_preferences_message_text_color</td><td>#212121</td></tr>
    <tr><td>Color of the background for the privacy preferences screen</td><td>ghostery_preferences_optin_background_color</td><td>#FFFFFF</td></tr>
    <tr><td>Color of the Opt-in subtext on the privacy preferences screen</td><td>ghostery_preferences_optin_subtext_color</td><td>#616161</td></tr>
    <tr><td>Color of the Opt-in main text and the On and Off control text on the privacy preferences screen</td><td>ghostery_preferences_optin_text_color</td><td>#212121</td></tr>
    <tr><th>From file:</th><td colspan="2">.../res/values/ghostery_strings.xml</td></tr>
    <tr><td>Height of the category divider on the privacy preferences screen</td><td>ghostery_list_category_divider_height</td><td>1</td></tr>
    <tr><td>Height of the list-item divider on the privacy preferences screen</td><td>ghostery_list_divider_height</td><td>0</td></tr>
    <tr><td>Max times to display the Implied consent flow in a 30-day period</td><td>ghostery_ric_max_default</td><td>3</td></tr>
    <tr><td>Consent flow dialog opacity (Only used on Android Honeycomb and higher. Must be an integer from 0 to 100. For a value < 100, the dialog will be full screen width.)</td><td>ghostery_ric_opacity</td><td>100</td></tr>
    <tr><td>Max times to display the Implied consent flow in an app session</td><td>ghostery_ric_session_max_default</td><td>1</td></tr>
    <tr><td>Text of the close button in the Implied Consent flow dialog</td><td>ghostery_dialog_button_close</td><td>Close</td></tr>
    <tr><td>Text of the Accept button in the Explicit Consent flow dialog</td><td>ghostery_dialog_button_consent</td><td>Accept</td></tr>
    <tr><td>Text of the Decline button in the Explicit Consent flow dialog</td><td>ghostery_dialog_button_decline</td><td>Decline</td></tr>
    <tr><td>Text of the Manage Preferences button in the Explicit Consent flow dialog</td><td>ghostery_dialog_button_preferences</td><td>Manage Preferences</td></tr>
    <tr><td>Text of the message in the Explicit Consent flow dialog</td><td>ghostery_dialog_explicit_message</td><td>Our app uses technologies so that we, and our partners, can remember you and understand how you use our app. To see a complete list of these technologies and to explicitly tell us whether they can be used on your device, click on the \"Manage Preferences\" button below. To give us your consent, click on the \"Accept\" button.</td></tr>
    <tr><td>Text of the title in both the Explicit Consent and Implied consent flow dialogs</td><td>ghostery_dialog_header_text</td><td>We Care About Your Privacy</td></tr>
    <tr><td>Text of the message in the Implied Consent flow dialog</td><td>ghostery_dialog_implicit_message</td><td>Our app uses technologies so that we, and our partners, can remember you and understand how you use our app. To see a complete list of these technologies and to tell us whether they can be used on your device, click on the \"Manage Preferences\" button below. Further use of this app will be considered consent.</td></tr>
    <tr><td>Text of the Please-wait message that is displayed whild downloading the app-notice preferences</td><td>ghostery_dialog_pleaseWait</td><td>Please Wait...</td></tr>
    <tr><td>Text of the description section near the top of the privacy preferences screen</td><td>ghostery_preferences_description</td><td>Our company with help from our partners, collects data about your use of this app. We respect your privacy and if you would like to limit the data we collect please use the control panel below. To find out more about how we use data please visit our privacy policy.</td></tr>
    <tr><td>Text of the learn-more paragraph above the learn-more link on the vendor detail screen</td><td>ghostery_preferences_detail_learnmore</td><td>To learn more about how we collect and use information for mobile apps, please visit:</td></tr>
    <tr><td>Text used in the case there is no learn-more link provided</td><td>ghostery_preferences_detail_learnmore_not_provided</td><td>Privacy Policy Not Provided</td></tr>
    <tr><td>Text for the header above the vendor-info section of the vendor detail screen </td><td>ghostery_preferences_detail_trackerinfo</td><td>Tracker Info</td></tr>
    <tr><td>Text that is displayed when no vendors are available for display on the privacy preferences screen</td><td>ghostery_preferences_empty_list</td><td>The list of trackers could not be loaded. Please ensure you have internet access, and try again later.</td></tr>
    <tr><td>Title text on the privacy preferences screen</td><td>ghostery_preferences_header</td><td>Manage Preferences</td></tr>
    <tr><td>Text of the Opt-in subtext on the privacy preferences screen</td><td>ghostery_preferences_optin_subtext</td><td>To all trackers listed below.</td></tr>
    <tr><td>Text of the Opt-in main text on the privacy preferences screen</td><td>ghostery_preferences_optin_text</td><td>Opt In</td></tr>
    <tr><td>Text for the select-all control on the privacy preferences screen</td><td>ghostery_preferences_select_all</td><td>All</td></tr>
    <tr><td>Text for the select-none control on the privacy preferences screen</td><td>ghostery_preferences_select_none</td><td>None</td></tr>
    <tr><td>Header text on the vendor detail screen</td><td>ghostery_tracker_detail_title</td><td>Tracker Detail</td></tr>
    <tr><td>Title text of the learn-more screen</td><td>ghostery_tracker_learnmore_title</td><td>Learn More</td></tr>
    </table>

  * For example:

  ```xml
ghostery_preferences_description
    From "Our company with help from..."
    To "(YourCompanyName) with help from..."
ghostery_dialog_header_text
    From "We Care About Your Privacy"
    To "(YourCompanyName) Cares About Your Privacy"
  ```

  * Customization examples for AAR WaterDrop strings:
  ```xml
<string name="ghostery_dialog_explicit_message">The WaterDrop app uses technologies so that we, and our partners, can remember you and understand how you use our app. To see a complete list of these technologies and to explicitly tell us whether they can be used on your device, click on the \"Manage Preferences\" button below. To give us your consent, click on the \"Accept\" button.</string>
<string name="ghostery_dialog_header_text">WaterDrop Cares About Your Privacy</string>
<string name="ghostery_dialog_implicit_message">The WaterDrop app uses technologies so that we, and our partners, can remember you and understand how you use our app. To see a complete list of these technologies and to tell us whether they can be used on your device, click on the \"Manage Preferences\" button below. Further use of this app will be considered consent.</string>
<string name="ghostery_preferences_description">WaterDrop with help from our partners, collects data about your use of this app. We respect your privacy and if you would like to limit the data we collect please use the control panel below. To find out more about how we use data please visit our privacy policy.</string>
  ```

3. Edit layouts in …\res\layout\*.xml

    **View** | **File to customize**
    ------------------- | --------------------------------
    Tracker Detail activity | ghostery_activity_tracker_detail.xml
    Tracker List activity | ghostery_activity_tracker_list.xml
    Tracker List activity | ghostery_explicitinfo_dialogfragment.xml
    Explicit Consent flow dialog | ghostery_explicitinfo_dialogfragment.xml
    Tracker Learn More fragment | ghostery_fragment_learn_more.xml
    Tracker Detail fragment | ghostery_fragment_tracker_detail.xml
    Implied Consent flow dialog | ghostery_impliedinfo_dialogfragment.xml
    Tracker list item | ghostery_tracker_list_item.xml

4. Edit or add any desired language/localization files. 
  1. Edit the English version of the SDK strings here: …\res\values\values.xml
  2. Edit the other existing language strings here: …\res\values-##\values-##.xml
    * Where "##" is the two-character language indicator.
  3. Add folders and XML files for additional languages in this format: …\res\values-##\values-##.xml
    * Where "##" is the two-character language indicator.
5. Zip the contents of the unzipped AAR back into a ZIP file
6. Copy that ZIP file back to libs folder it came from.
7. Rename this new ZIP file to AppNoticeSDK.aar
8. Use this customized AppNoticeSDK.aar file in your app as per the  instructions above.
