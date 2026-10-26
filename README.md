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
- **LineageOS** and **TheMuppets**: reference vendor trees.
