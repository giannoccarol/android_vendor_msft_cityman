# Cityman camera blobs

The rear camera uses the Qualcomm camera daemon from the Nexus 6P (angler),
MSM8994 with VFE 4.6, instead of the Nexus 5X (bullhead) daemon this tree
inherited, which has no VFE 4.6 ISP modules.

- Source: Google factory image `angler-opm7.181205.001-factory-b75ce068.zip`
  (SHA-256 `b75ce068f23a0e793805f80fccbc081eca52861ef5eb080c47f502de4c3f9713`), `vendor.img`.
- 131 files: `mm-qcamera-daemon`, `libmmcamera2_*`, ISP modules
  (`*40`, `*44`, `*46`), JPEG/QOMX, actuator/EEPROM/flash libraries and the
  `imx377`/`imx179` chromatix. They overwrite the bullhead copies and the
  48 files bullhead lacks are listed at the end of `cityman-vendor.mk`.
  Bullhead-only camera files are kept; the angler sensor module does not
  load them.
- The Android 13 camera branch deliberately omits the old IMX377-derived
  `libmmcamera_imx230.so` probe library. Its register programs were for another
  sensor and it could enumerate the IMX230 without delivering real frames.
  It is removed from both this branch and the product copy list.
- The kernel branch is `lineage-20-cityman-camera`, created directly from
  `lineage-20-cityman`. Its camera driver is aligned with this angler daemon.
- The device camera branch now supplies a new source IMX230 SMIA++ module,
  with an independently specified interface tested against the actual angler
  consumer, including the Android-built shared object. It is selected only by
  `CITYMAN_CAMERA_SMIA_EXPERIMENTAL=true`. No new sensor binary is stored in
  this vendor repository. Actual phone streaming and calibration are untested.
  Front Ducati has no validated module. No Motorola, Nubia or HTC camera
  libraries were added; see the device's `camera-sensors/imx230/ABI.md`.

Per-file hashes are in `CAMERA-CITYMAN.sha256`.

`liboemcamera.so` is the angler file with one change: its `libsensor.so`
dependency is renamed to `libsensor_vendor.so` (patchelf 0.17.2,
`--replace-needed`), because the Android 13 libsensor no longer has
`SensorManager::createEventQueue(String8, int)`. The original SHA-256 is
`77983655cd0beb733363804378c68a70c84d4e290cddadbbed044f80aca2776e`.
