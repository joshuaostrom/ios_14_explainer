# iOS 14 et tu: An Ad Tech Explainer

This brief explainer provides some thoughts on two of the changes in iOS 14 as it relates to Ad Tech.  In both cases a few exchange SDKs are already positioned for a seamless migration to iOS 14.

# IDFA
 - Existing apps will report IDFA of '00000000-0000-0000-0000-000000000000'
 - No prompt is given for existing apps, IDFA is zeros by default
 - 3 requirements must be met for IDFA to be passed
    - App is updated to include requestTrackingAuthorization
    - User installs / updates app 
    - User clicks “Allow Tracking”
  - IDFA resets now occurs when tracking is toggled off for the only app using tracking
  - Some exchange SDKs do provide fallback IDFA values when IDFA is unavailable

# IDFA Action Items
  - App publishers should update apps to include requestTrackingAuthorization
  - Exchange SDKs could provide fallback IDFA values.
  Sample code:

```
// App can call `getAdvertisingId()` when the IDFA is needed.  If IDFA is unavailable it'll grab a persistent user id for this app install 

func getAdvertisingId() -> String {
  if ATTrackingManager.trackingAuthorizationStatus == ATTrackingManager.AuthorizationStatus.authorized{
  // IDFA is enabled
    return ASIdentifierManager.shared().advertisingIdentifier.uuidString
  } else {
    // IDFA disabled return appUserId
    return self.getAppUserId()
  }
}

func getAppUserId() -> String {
  let defaults = UserDefaults.standard 
  if let uid = defaults.object(forKey:"uid") as? String ?? nil {
    return uid
  } else {
    let newid = UUID().uuidString
    defaults.set(newid, forKey: "uid")
    return newid
  }
}
```
Note the fallback IDFA will be different for every application.  Supplying the `suiteName:` to the user defaults would allow a common identifier across apps via App groups. To honor the intent of not tracking across apps we are just using `UserDefaults.standard`.

# Location
  - Default is to allow precise
  - User can manually turn on approximate via Settings->App
  - Approximation state can be detected
    - ```
      CLLocationManager().accuracyAuthorization == CLAccuracyAuthorization.fullAccuracy
  - Better yet, horizontalAccuracy reflects the approximation:

    - ```
      func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        let userLocation :CLLocation = locations[0] as CLLocation
        print("latitude: \(userLocation.coordinate.latitude)")
        print("longitude: \(userLocation.coordinate.longitude)")
        print("horizontalAccuracy: \(userLocation.horizontalAccuracy)")

  - Open RTB 2.4 onward support accuracy on geo object.
  - Some exchange SDKs are passing the accuracy today
 
# Location action items

   - More exchanges should update SDKs to pass the accuracy parameter as supported in Open RTB 2.4+
     - Declaring the accuracy allows the demand side to filter the data for the desired targeting tactics
