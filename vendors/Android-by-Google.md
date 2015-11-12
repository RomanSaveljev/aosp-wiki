This is basically an AOSP without customization. Nexus devices run it.

Google runs a [dedicated](https://source.android.com/) web-site with some information on:

* [Porting](https://source.android.com/devices/index.html)
* [Getting compatible](https://source.android.com/compatibility/index.html)

From [another](https://developers.google.com/android/) place Nexus factory [images](https://developers.google.com/android/nexus/images) and [proprietary blobs](https://developers.google.com/android/nexus/drivers) can be fetched.

## Notes on building Nexus firmware

Synchronize repo manifest and drop all binary blobs into the workspace. Each blob unpacks into `extract_..` script. Running it asks to accept the license and will copy vendor files where they should be. Proceed with `lunch` and `make`.

Skipping binary blobs still creates flashable images, but the device does not start properly with them.