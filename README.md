## OptiCache-Alpha Implementation Doc

**Steps Included in this guide are**
1. Install AdsCache Library
2. Set up the updated Utility Class
3. Add Your AdUnit Ids
4. Update source code for the new show method
5. Define these parameters in Firebase Remote Config in the project being used (Optional)
6. Test

**Resources**
**1. Github Link
2. ArrFile**
**3. AppBrodaAdUnitHandler.kt**

> 💡 OptiCache curenlty only supports Interstitial and Rewarded Ads

## 1. Install AdsCache library

AdsCache Library depends on GMA SDK, update the build.gradle file and add the by following
this guide and using the .arr file shared above.

    Implementation files('<PATH_TO_FILE>\\adscache-11.09.aar')

## 2. Set up the updated Utility Class
Download the Kotlin file and add/replace it in your project. This Utility class,
`AppBrodaAdUnitHandler` will be used to cache and show ads.

# Option A: If Remote Config is already configured.

**Save Ad Units Using Utility Class**

Make sure you have followed the official Remote Config guide up to    Step 6. In the same activity as the remote config (**MainActivity.kt** in  this example below) Import the `AppBrodaAdUnitHandler` class and call    the `AppBrodaAdUnitHandler.fetchAndSaveAdUnits()` after the values have  been successfully fetched and activated. New changes have been    highlighted in the code below using the ***Step 6*** from the official guide as an example.

```kotlin
remoteConfig.fetchAndActivate()
    .addOnCompleteListener(this) { task ->
        if (task.isSuccessful) {
            val updated = task.result
            Log.d(TAG, "Config params updated: $updated")
            Toast.makeText(
                this,
                "Fetch and activate succeeded",
                Toast.LENGTH_SHORT
            ).show()
        } else {
            Toast.makeText(
                this,
                "Fetch failed",
                Toast.LENGTH_SHORT
            ).show()
        }

        displayWelcomeMessage()
        AppBrodaAdUnitHandler.fetchAndSaveAdUnits(this)
    }
```

##  Set the minimum fetch interval for remote config to 15mins
Set the minimum interval to 15 mins this is is crucial to maintain the compatibility.
```kotlin
val mFirebaseRemoteConfig: FirebaseRemoteConfig = Firebase.remoteConfig

mFirebaseRemoteConfig.setConfigSettingsAsync(remoteConfigSettings {
    minimumFetchIntervalInSeconds = 900
})
// Set minimum fetch interval to 0 during testing
mFirebaseRemoteConfig.fetchAndActivate()

```

# Option B: If Remote Config has not been configured

**Save Placements Using Utility Class**

To initialize the above utility class, Import the AppBrodaPlacementHandler class in
**MainActivity.kt** and call the
`AppBrodaPlacementHandler.initRemoteConfigAndSavePlacements()` The new changes have been highlighted in the code below.

```kotlin
package ...

import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import com.google.android.gms.ads.MobileAds

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        MobileAds.initialize(this) {
            AppBrodaAdUnitHandler.initRemoteConfigAndSaveAdUnits(this)
        }
    }
}
```


> 🛠️TESTING : To make testing easier, temporarily set the minimum fetch
> interval to 0s. **Remember to set the minimum interval back to 15 mins (
> 900 seconds ) before releasing the app.**


## 3. Add Your AdUnit Ids
For OptiCache to properly configure your adUnits you have to also add them to the
Config by using the function `pushAdUnitToConfig(...)` inside the `AppbrodaAdUnitHandler.kt`

> 💡 OptiCache curenlty only supports Interstitial and Rewarded Ads

```kotlin
pushAdUnitToConfig(
    YOUR_ADUNIT_ID,                // adUnitId
    YOUR_ADUNIT_FORMAT,            // format
    adUnitsConfigInterstitialQueueSize,
    adUnitsConfigInterstitialLoadInterval
)
```



## 4. Update source code for the new show method
With OptiCace Impl the logic for the following is not required anymore -

 - Pre-fetching the Ads  
 - Loading an Ad for a configured ad unit    
 - Handling the expired Ads

Call the showAd method wherever you want to show an Ads and pass the AdUnit Id to
the function.
The AdUnit Id being used should have already been added in ***step 3***.

***Example***

```kotlin
showAdButton.setOnClickListener {
    AppBrodaAdUnitHandler.showAd(this, "com_example_appbrodasampleapp_interstitialAds")
}

```


## 5. Define these parameters in Firebase Remote Config in the project being used (Optional)
- `AdsConfig_Interstitial_QueueSize` ( Number, default 2 )
- `AdsConfig_Interstitial_LoadInterval` ( Number, default 1800 )
- `AdsConfig_Rewarded_QueueSize` ( Number, default 2 )
- `AdsConfig_Rewarded_LoadInterval` ( Number, default 3600 )

## 6. Test
Your ads should now be cached optimally. For the best test case you can use Adx and
Admob Unit Ids one after the other. The ads that have been already been cached
should still be shown if you turn off your network. Once an ad is shown it is removed
from the cache.

