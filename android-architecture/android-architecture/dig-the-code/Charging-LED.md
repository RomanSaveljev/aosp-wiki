[Different](http://androidxref.com/6.0.0_r1/xref/hardware/libhardware/include/hardware/lights.h#33) logical LED are distinguished by Android:

* backlight
* keyboard
* buttons
* battery
* notifications
* attention

They are logical, because they merely identify a LED type, but actual routing to the correct hardware is left to BSP. In CAF BSP same LED is used both for charging status and notifications. The actual PWM channels are controllable through sysfs nodes:

* `/sys/class/leds/red/brightness`
* `/sys/class/leds/green/brightness`
* `/sys/class/leds/blue/brightness`

The BSP writes these files in [lights.c](http://androidxref.com/6.0.0_r1/xref/hardware/qcom/display/msm8226/liblight/lights.c).

 In Java side they
are controlled from [LightsService#setLightLocked](http://androidxref.com/6.0.0_r1/xref/frameworks/base/services/core/java/com/android/server/lights/LightsService.java#96).
It calls to the [native](http://androidxref.com/6.0.0_r1/xref/frameworks/base/services/core/jni/com_android_server_lights_LightsService.cpp#106) part,
which goes into BSP. For battery LED it ends up calling
[set_light_battery](http://androidxref.com/6.0.0_r1/xref/hardware/qcom/display/msm8226/liblight/lights.c#set_light_battery).
The function is accessed through a hardware module `set_light` function pointer
member, which is assigned in [open_lights](http://androidxref.com/6.0.0_r1/xref/hardware/qcom/display/msm8226/liblight/lights.c#271).

Hardware module structure with function pointers is obtained in [init_native](http://androidxref.com/6.0.0_r1/xref/frameworks/base/services/core/jni/com_android_server_lights_LightsService.cpp#63).

The actual sys-node writing happens in [handle_speaker_battery_locked](http://androidxref.com/6.0.0_r1/xref/frameworks/base/services/core/jni/com_android_server_lights_LightsService.cpp#63).
On MSM there is no dedicated battery LED, so battery LED indication goes to the
same notification LED. The `handle_speaker_battery_locked` shows that if "battery"
LED is lit (its color is not black), then it will take precedence over other
notifications.

[BatteryService](http://androidxref.com/6.0.0_r1/xref/frameworks/base/services/core/java/com/android/server/BatteryService.java#741)
will control "battery" LED depending on battery status and events. The colors are
picked from the resource:

* `com.android.internal.R.integer.config_notificationsBatteryLowARGB`
* `com.android.internal.R.integer.config_notificationsBatteryMediumARGB`
* `com.android.internal.R.integer.config_notificationsBatteryFullARGB`

According to BSP code, to suppress battery LED indications one could configure all colors to black.

## Nexus 7 charging LED

This is a practical explanation example of why Nexus 7 (deb) never lits its notification LED on charging events.

According to traces [setLight_native](http://androidxref.com/6.0.0_r1/xref/frameworks/base/services/core/jni/com_android_server_lights_LightsService.cpp#106) exits prematurely, because `devices->lights[light]` (where `light` is [LIGHT_INDEX_BATTERY](http://androidxref.com/6.0.0_r1/xref/frameworks/base/services/core/jni/com_android_server_lights_LightsService.cpp#39)) is `NULL`.

It could be `NULL`, if previous hardware module opening would fail. The hardware module is implemented in BSP and happens to be `lights.$(TARGET_BOARD_PLATFORM)`. By calling in [this](android-build-system/hmm-commands/get-build-var) trick we can find out its value. It is `msm8960`.

Now, if we look into specific [lights.c](http://androidxref.com/6.0.0_r1/xref/hardware/qcom/display/msm8960/liblight/lights.c#284), then we can see there is no dedicated charging LED. Battery service attempts to control a charging LED state, but the calls are silently ignored.

Another example is [msm8226](http://androidxref.com/6.0.0_r1/xref/hardware/qcom/display/msm8226/liblight/lights.c#271) BSP implementation. It uses a single LED to act as both notification and charging logical LEDs.