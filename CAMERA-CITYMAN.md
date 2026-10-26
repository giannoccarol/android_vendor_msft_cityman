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
- A matching Lumia sensor module is still required. This is a development
  baseline, not a camera fix or a complete SMIA++ integration. Front Ducati
  has no validated module either. No Motorola, Nubia or HTC camera libraries
  have been added to replace the missing sensor module.

Per-file hashes are in `CAMERA-CITYMAN.sha256`.
