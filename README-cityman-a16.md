# Blob changes for Android 16 (lineage-23-cityman)

Android 16 always runs the full Treble linker configuration: vendor processes
see only `/vendor` libraries and the LLNDK set. Android 13 ran this device with
`ro.treble.enabled=false`, so several blobs still linked system libraries; and
the LineageOS `LD_SHIM_LIBS` mechanism no longer exists in bionic. The blobs are
therefore patched in place with `patchelf` (0.18, from
`prebuilts/extract-tools`), the way the OnePlus 2 tree does it in its
`extract-files.py`, and the missing libraries are installed as vendor variants
from `device.mk`.

| Blob | Change |
|---|---|
| `vendor/lib64/libril-qc-qmi-1.so` | `libaudioclient.so` → `libaudioclient_shim_vendor.so` (AudioSystem stubs, device `libshims/`) |
| `vendor/lib/liboemcamera.so` | drop `libgui.so`; `libsensor.so` → `libsensor_camera_stub.so` (device `libshims/`: a SensorManager with no sensors; the LineageOS `libsensor_vendor` lacks `SensorEventQueue::getFd()` and hung the daemon on Android 13); `libandroid.so` → `libsensorndkbridge.so` |
| `vendor/lib/libmmcamera2_imglib_modules.so` | drop `libgui.so`; `libsensor.so` → `libsensor_camera_stub.so` |
| `vendor/lib/libmmcamera2_stats_modules.so` | `libandroid.so` → `libsensorndkbridge.so` |
| `vendor/lib/libmmcamera_faceproc.so`, `libgoog_eis_armeabi-v7a.so`, `libgoog_rownr.so` | `libstdc++.so` → `libstdc++_vendor.so` |
| `vendor/lib{,64}/mediadrm/libwvdrmengine.so` | `libprotobuf-cpp-lite.so` → `libprotobuf-cpp-lite-3.9.1-vendorcompat.so`; add `libcrypto_shim.so` |
| `bin/ATFWD-daemon`, `bin/cnd`, `vendor/lib{,64}/libcne.so` | add `libcutils_shim.so` (was `TARGET_LD_SHIM_LIBS`) |
| `bin/netmgrd`, `bin/qmuxd` | add `libcityman_ds_target.so` (was an `LD_PRELOAD` in `init.cityman.rc`, inherited by the system `sh` they spawn, which cannot load vendor libraries) |
| `proprietary/bin/pm-service` | add `libutils-v33.so` (Android 16 libutils aborts on "RefBase used with stack pointer argument"; without pm-service the modem is never powered) |

Other changes:
- `libloc_api_v02.so` and `libloc_ds_api.so` move from `system/` to vendor: the
  vendor location blobs (`izat`, `lbs`, `slim_daemon`) need them.
- The IMS stack (daemons, `lib-ims*`, `lib-rcs*`, IMS JNI) is not installed:
  it was already disabled (no IMS APK, no VoLTE) and frees ~12 MB on the 248 MB
  vendor partition, which since Android 14 also holds the former VNDK libraries.
- `libllvm-qcom.so` is not installed (nothing loads it; it served the absent
  `libOpenCL.so`).
- Not handled yet: `libRSDriver_adreno.so` (needs the removed RenderScript
  system libraries), `bluetooth.default.so` and `nfc@1.0-impl` (unused).

Original files: the parent commit in git. Check any vendor ELF against the
Android 16 namespace with the scan used for this list (NEEDED and undefined
symbols resolved against `/vendor` + `system/etc/llndk.libraries.txt`).

## Credits

The patch list follows the OnePlus 2 Android 16 tree by Davi (davigamer987),
[device_oneplus_oneplus2](https://github.com/davigamer987/device_oneplus_oneplus2)
`lineage-23.2`, `extract-files.py`; compatibility libraries from LineageOS
`hardware/lineage/compat`.
