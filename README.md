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
- Venus (video codecs): `venus.mdt`, `venus.b00`-`b04` split from the Cityman
  `qcvss8994.mbn`, and `venus.mbn`, the unsplit image (see
  `Documentation/arm64/cityman-venus.txt` in the kernel). The angler/bullhead
  Venus image they replace fails authentication on this phone, which left
  every hardware video codec unavailable.
- Modem MBA: `mba.b00`, the 367,824-byte payload (offset `0x1000`) of the
  Cityman `qcdsp1v28994.mbn` (build AB6A9EB9, sha256 of the `.mbn`
  `6b75a202c7e05f042a50bfb98e5abae563e5129934146489d2d29d84a2cee60a`).
  The modem partition written by the installer holds another MBA (build
  AC1B8953, md5 `07d53b0364c8ccb58c49a4522daa5ebe`) that the modem boot ROM
  rejects (`RMB_PBL_STATUS ef0b000b`, "MBA boot failed"), so the modem never
  started and there was no mobile network. ueventd searches `/vendor/firmware`
  before `/firmware/image`, so this one is used without rewriting the modem
  partition.

## Modem image shipped in /system/etc/firmware

The installer's `modem.img` is the Lumia 950 (talkman, MSM8992) modem: its
MBA and its `modem.mdt` carry `HW_ID 009690E1 0045 0000` (Nokia CA
`0000052A`), while the 950 XL boot ROM only accepts MSM8994 images
(`HW_ID 009400E1 0045 0000`, CA `00000526`, like the Cityman ADSP and Venus).
Both are the same modem build, `MPSS.BO.2.5.c4.3-00024-M8994FAAAANAZM-1`
(2016-06-02); only the signature differs. On some phones the Cityman MBA
rejected the modem headers of the partition (`MBA returned error -3 for
headers`).

So the ROM ships the whole Cityman modem: `modem.mdt` and `modem.b00`-`b22`
split from the Windows `qcdsp28994.mbn` (sha256
`8a341eb2d92d9ee6c2430d7d6316d54866bc74535b46341d6e9b355711db4ac9`;
`modem.mdt` = `b00` + `b01`, one `bNN` per program header with data; all 17
segments match the signed hash table), with the same `mba.b00`. They are in
`/system/etc/firmware` because the vendor partition has no room for 46 MB, and
ueventd searches `/etc/firmware` first. Only `modem_pr/mcfg` (carrier
configurations) still comes from the modem partition.

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
  the Bluetooth firmware, the Cityman `qcadsp8994.mbn`, `qcvss8994.mbn` and
  `qcdsp1v28994.mbn` (MBA) and `qcdsp28994.mbn` (modem image, commit `f489e27`).
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
