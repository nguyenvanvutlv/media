# Building AV1 Decoder for Android (16KB Page Size Support)

This document describes the steps to build the AV1 decoder module as an AAR file with support for 16KB page size.

## Prerequisites

1.  **Android NDK**: NDK r27 or higher is required for 16KB page size support.
2.  **Build Tools**: `meson`, `ninja`, `nasm`, and `cmake` must be installed on your system.
3.  **Source Code**: Clone this repository and navigate to the `libraries/decoder_av1` directory.

## Step 1: Set Environment Variables

```bash
# Path to the project root
PROJECT_ROOT="$(pwd)"
AV1_MODULE_PATH="${PROJECT_ROOT}/libraries/decoder_av1/src/main"
NDK_PATH="/Users/admin/Library/Android/sdk/ndk/27.0.12077973" # Update this path if necessary
HOST_PLATFORM="darwin-x86_64" # Use linux-x86_64 for Linux
```

## Step 2: Fetch External Dependencies

Navigate to the JNI directory and clone the required libraries:

```bash
cd "${AV1_MODULE_PATH}/jni"
git clone https://github.com/google/cpu_features
git clone https://code.videolan.org/videolan/dav1d.git
```

## Step 3: Ensure 16KB Page Size Alignment

The 16KB page size support is enabled by adding the following linker flag in `libraries/decoder_av1/src/main/jni/CMakeLists.txt`:

```cmake
target_link_options(dav1dJNI
                    PRIVATE "-Wl,-z,max-page-size=16384")
```

*(This is already included in the current version of the code).*

## Step 4: Build dav1d Native Library

Run the `build_dav1d.sh` script to build the static libraries for all ABIs:

```bash
cd "${AV1_MODULE_PATH}/jni"
./build_dav1d.sh \
  "${AV1_MODULE_PATH}" \
  "${NDK_PATH}" \
  "${HOST_PLATFORM}"
```

## Step 5: Build the AAR

Return to the project root and use Gradle to assemble the release AAR:

```bash
cd "${PROJECT_ROOT}"
./gradlew :libraries-decoder_av1:assembleRelease
```

The resulting AAR file will be located at:
`libraries/decoder_av1/build/outputs/aar/decoder_av1-release.aar`

