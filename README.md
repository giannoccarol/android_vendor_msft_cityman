# Lumia 950 XL (cityman) vendor tree, LineageOS 18.1

Proprietary files for the cityman ROM built from
`giannoccarol/android_device_msft_cityman` (branch `lineage-18.1-cityman`).
Listed in `cityman-vendor.mk`; the camera stack is described in
`CAMERA-CITYMAN.md`.

## Firmware shipped in /vendor/firmware

- Wi-Fi: `qwlan30.bin` (Android SGMT), the cityman `bdwlan30.bin` board file,
  `wlan/qca_cld/WCNSS_qcom_cfg.ini`.
- Bluetooth: `nvm_tlv_3.0/3.2.bin`, `rampatch_tlv_3.0/3.2.tlv`.
- ADSP: `adsp.mdt`, `adsp.b00`-`b04`, `b06`, `b08`-`b12`, `b14`, `b16`, `b17`,
  split from the Cityman `qcadsp8994.mbn` (see
  `Documentation/arm64/cityman-adsp.txt` in the kernel).

No per-device data is shipped: MAC addresses and the modem EFS stay in the
phone's `persist` and modem partitions.

## Rear camera tuning (Android 13)

The rear IMX230 uses the angler IMX377 chromatix. Ten of those libraries
(`libchromatix_imx377_common.so` and the `cpp_*` ones) are modified for
cityman ("v4-combo"):

- lens shading: luminance corrected at 75% and colour shading at half, the
  Lumia rule found in the stock Windows camera calibration; full IMX377
  shading gave purple corners and amplified corner noise;
- CPP wavelet denoise: luma noise profile x1.4, chroma x1.6, because the
  IMX230 1.12 um pixels are noisier than the IMX377 ones.

The changes are plain scaling of the existing tables; no Lumia calibration
data is shipped. `CAMERA-CITYMAN.sha256` lists the modified files.

`libmmcamera2_sensor_modules.so` (angler) has one change: the unused `imx258`
entry of its compiled sensor list is renamed `vs6955`, so the daemon loads
the front VS6955 module from the device tree (`camera-sensors/vs6955`).

## Credits

- **Android4Lumia950** (djokcho, EpicLPer, j0sh1x and the community): the
  talkman vendor tree this one is based on (`lineage-18.1-julian`) and
  `Android4Lumia950/firmware-microsoft-cityman`, the source of `bdwlan30.bin`,
  the Bluetooth firmware and the Cityman `qcadsp8994.mbn`.
- **Google / Qualcomm**: the angler and bullhead 8.1 blobs (camera daemon
  stack and other vendor libraries).
- **shadowofleaf96** (`shadowofleaf96/android_vendor_msft_cityman`,
  `9e57307`): the Cityman ACDB calibration files
  (`etc/acdbdata/cityman/Cityman_*_cal.acdb`), which match the Cityman ADSP
  firmware; the bullhead ACDB makes the ADSP refuse the audio streams.
- **OnePlus / Qualcomm**, via **davigamer987**
  (`davigamer987/vendor_oneplus_oneplus2`, branch `lineage-22.2`, `1abfce3`):
  the MSM8994 `perfd` daemon, `perf-profile1`-`5.conf` and
  `libqti-perfd-client.so` (32 and 64 bit) from the OnePlus 2 (OxygenOS
  6.0.1). The LineageOS 20 OnePlus 2 port by **mukul2259** / **_undead**
  (`mukul2259/experimental_device_oneplus_oneplus2`, `6cdfa90`) uses the same
  blobs with the QTI power HAL. They replace the open-source
  `libqti-perfd-client` stub of the device tree, so power HAL hints and
  thermal-engine profiles reach a real daemon.
- **LineageOS** and **TheMuppets**: reference vendor trees.
- **Lorenzo Giannoccaro** (giannoccarol): reverse engineering of the Lumia
  950 XL Windows camera calibration (DCC) behind the rear lens shading and
  denoise changes, and the colour A/B tests on the phone.
