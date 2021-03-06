---
title: "Android Prebuilts"
description: ""
date: 2020-02-20T20:50:46+01:00
draft: false
---

Sometimes, you want to use a library or an APK file that someone else already
built and maybe signed for you. Android calls these "prebuilts".

Current supported prebuilt types:
- Shared (C/C++) library
- Static (C/C++) library
- Binary (executable)
- Java library
- `/etc/` file

See [build/soong/androidmk/cmd/androidmk/android.go][androidmk-q]:
```
var prebuiltTypes = map[string]string{
    "SHARED_LIBRARIES": "cc_prebuilt_library_shared",
    "STATIC_LIBRARIES": "cc_prebuilt_library_static",
    "EXECUTABLES":      "cc_prebuilt_binary",
    "JAVA_LIBRARIES":   "java_import",
    "ETC":              "prebuilt_etc",
}
```
Supported on current master, as of 2020-02-20:  
[build/soong/androidmk/androidmk/android.go][androidmk-q]:
```
var prebuiltTypes = map[string]string{
    "SHARED_LIBRARIES": "cc_prebuilt_library_shared",
    "STATIC_LIBRARIES": "cc_prebuilt_library_static",
    "EXECUTABLES":      "cc_prebuilt_binary",
    "JAVA_LIBRARIES":   "java_import",
    "APPS":             "android_app_import",
    "ETC":              "prebuilt_etc",
}
```

<div class="message">
<code>PREBUILT_SHARED_LIBRARY</code> is deprecated. Define
<code>LOCAL_MODULE_CLASS</code> and call <code>BUILD_PREBUILT</code> instead.
</div>

### Binary
Using `make` syntax:
```
include $(CLEAR_VARS)
LOCAL_MODULE := myexec
LOCAL_SRC_FILES := myexec
LOCAL_MODULE_CLASS := EXECUTABLES
LOCAL_MODULE_TAGS := optional
# Optional:
LOCAL_MULTILIB := 64
LOCAL_PROPRIETARY_MODULE := true
LOCAL_SHARED_LIBRARIES := libmyexample
include $(BUILD_PREBUILT)
```
and in `blueprint` format:
```
cc_prebuilt_binary {
    name: "myexec",
    srcs: ["myexec"],
    // Optional:
    vendor: true,
    relative_install_path: "hw",
    // Soong supports the same extras as for regular cc_binary:
    shared_libs: [
        "libmyexample",
    ],
    //vintf_fragments: ["vendor.acme.myexec@1.0-service.xml"],
    //init_rc: ["vendor.acme.myexec@1.0-service.rc"],
}
```

### (Shared) library
Using `make` syntax:
```
include $(CLEAR_VARS)
LOCAL_MODULE := libmyexample
LOCAL_SRC_FILES := $(LOCAL_MODULE).so
# For shared:
LOCAL_MODULE_CLASS := SHARED_LIBRARIES
# For static:
#LOCAL_MODULE_CLASS := STATIC_LIBRARIES
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_SUFFIX := .so
# Optional:
LOCAL_MULTILIB := 64
LOCAL_PROPRIETARY_MODULE := true
include $(BUILD_PREBUILT)
```
and in `blueprint` format:
```
cc_prebuilt_library_shared {
    name: "libmyexample",
    srcs: ["libmyexample.so"],
    // Optional:
    compile_multilib: "64",
    proprietary: true,
    strip: {
        none: true,
    },
}
// For static:
//cc_prebuilt_library_static {
```

### App
Using `make` syntax:
```
include $(CLEAR_VARS)
LOCAL_MODULE := MyApp
LOCAL_SRC_FILES := MyApp.apk
LOCAL_CERTIFICATE := platform
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_CLASS := APPS
LOCAL_MODULE_SUFFIX := .apk
# Optional:
LOCAL_PRIVILEGED_MODULE := true
LOCAL_PROPRIETARY_MODULE := true
include $(BUILD_PREBUILT)
```
`blueprint` support for `android_app_import` is in current master (as of
2020-02-20), will be available in Android R:
```
android_app_import {
    name: "MyApp",
    // Make sure the build system doesn't try to resign the APK
    dex_preopt: {
        enabled: false,
    },
    apk: "MyApp.apk",
    presigned: true,
}
```

### /etc file
Using `make` syntax:
```
include $(CLEAR_VARS)
LOCAL_MODULE := privapp-permissions-myapp.xml
LOCAL_SRC_FILES := privapp-permissions-myapp.xml
LOCAL_MODULE_CLASS := ETC
LOCAL_MODULE_TAGS := optional
# Optional, else just goes into /system/etc
LOCAL_MODULE_PATH := $(TARGET_OUT_VENDOR)/etc/permissions
LOCAL_PROPRIETARY_MODULE := true
include $(BUILD_PREBUILT)
```
and in `blueprint` format:
```
prebuilt_etc {
    name: "privapp-permissions-myapp.xml",
    src: "privapp-permissions-myapp.xml",
    // Optional
    sub_dir: "permissions",
    vendor: true,
}
```

# Converting
As always, you can use [the androidmk tool][androidmk-post] to convert from
`make` to `blueprint` syntax - not the other way around, though. Take a look
into the source code of `build/make` and `build/soong` to find out equivalents
if you must.

[androidmk-q]: https://android.googlesource.com/platform/build/soong/+/refs/tags/android-10.0.0_r29/androidmk/cmd/androidmk/android.go#931
[androidmk-master]: https://android.googlesource.com/platform/build/soong/+/988414c2cf6bfb868df7d402e0bf825d6fd44cc8/androidmk/androidmk/android.go#934
[androidmk-post]: {{< ref "blueprints.md" >}}
