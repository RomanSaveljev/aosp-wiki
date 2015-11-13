## Preamble

Android will show "Unsafe volume level warning", when audio is listened to through a headset and its loudness goes above certain level.

![Unsafe volume warning example](http://cdn3.howtogeek.com/wp-content/uploads/2014/08/android-disable-unsafe-volume-warning.png)

## Implementation

The [SafetyWarningDialog](http://androidxref.com/6.0.0_r1/xref/frameworks/base/packages/SystemUI/src/com/android/systemui/volume/SafetyWarningDialog.java)
is responsible of showing the warning. The
[AudioService#postDisplaySafeVolumeWarning](http://androidxref.com/6.0.0_r1/xref/frameworks/base/services/core/java/com/android/server/audio/AudioService.java#5862) appears to be an ultimate trigger of the warning.

There is a unsafe volume music limit constant defined as [UNSAFE_VOLUME_MUSIC_ACTIVE_MS_MAX](http://androidxref.com/6.0.0_r1/xref/frameworks/base/services/core/java/com/android/server/audio/AudioService.java#5445).
It is set to repeat the warning message once in every 20 hours of "unsafe" volume exposure.

The warning can be disabled altogether by manipulating [AudioService#mSafeMediaVolumeState](http://androidxref.com/6.0.0_r1/xref/frameworks/base/services/core/java/com/android/server/audio/AudioService.java#5437). Before its declaration is a good comment explaining the rules. It is a country-specific regulation and warning manifestation depends on the country code found by a network service.