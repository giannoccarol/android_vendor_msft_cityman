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
- `libmmcamera_imx230.so` (SHA-256 `0bea6822…`) is a probe library for
  the rear Sony IMX230, derived from the bullhead `libmmcamera_imx377.so`
  by `tools/cityman-camera/make-imx230-lib.py` in the kernel tree. It
  carries IMX377 register tables: Android lists the camera, but the sensor
  does not stream yet (SOF freeze).
- The kernel must be giannoccarol/android_kernel_mmo_msm8994
  `lineage-18.1-cityman-camera`, whose camera_v2 driver matches this
  daemon's ioctls.

Per-file hashes are in `CAMERA-CITYMAN.sha256`.
