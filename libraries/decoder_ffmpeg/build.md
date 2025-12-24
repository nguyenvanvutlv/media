# Hướng dẫn Build FFmpeg Decoder cho Android (16KB Page Size)

Tài liệu này hướng dẫn chi tiết cách build module FFmpeg decoder với hỗ trợ **16KB page size**, tương thích với Android 15 và các thiết bị phần cứng mới.

## 1. Yêu cầu tiên quyết

*   **Hệ điều hành**: macOS hoặc Linux.
*   **Android NDK**: Bắt buộc phiên bản **r27** trở lên (Để hỗ trợ căn chỉnh ELF 16KB).
*   **Công cụ**: `git`, `make`, `yasm`, `nasm`, `cmake`, `ninja`.

## 2. Thiết lập môi trường

Mở terminal và thiết lập các biến môi trường sau (thay đổi đường dẫn phù hợp với máy của bạn):

```bash
# 1. Đường dẫn đến thư mục gốc của project
cd "path"

# 2. Đường dẫn module FFmpeg trong project
FFMPEG_MODULE_PATH="$(pwd)/media/libraries/decoder_ffmpeg/src/main"
# 3. Đường dẫn Android NDK (phải là r27+)
NDK_PATH="/Users/admin/Library/Android/sdk/ndk/27.0.12077973"
# 4. Nền tảng host (darwin-x86_64 cho Mac, linux-x86_64 cho Linux)
HOST_PLATFORM="darwin-x86_64"
# 5. Android ABI version (minSdk)
ANDROID_ABI=21
```

## 3. Chuẩn bị mã nguồn FFmpeg

Tải mã nguồn FFmpeg và liên kết vào project:

```bash
# Clone FFmpeg bản release 6.0
git clone --depth 1 --branch release/6.0 git://source.ffmpeg.org/ffmpeg ffmpeg_source
FFMPEG_PATH="$(pwd)/ffmpeg_source"

# Tạo liên kết (link) vào module JNI
cd "${FFMPEG_MODULE_PATH}/jni"
ln -s "$FFMPEG_PATH" ffmpeg
```

## 4. Build thư viện Native (.a)

Sử dụng script `build_ffmpeg.sh`. Script này đã được cấu hình cờ `-Wl,-z,max-page-size=16384`.

```bash
cd "${FFMPEG_MODULE_PATH}/jni"

# Chọn các bộ giải mã cần dùng
ENABLED_DECODERS=(vorbis opus flac)

# Thực thi build
./build_ffmpeg.sh \
  "${FFMPEG_MODULE_PATH}" \
  "${NDK_PATH}" \
  "${HOST_PLATFORM}" \
  "${ANDROID_ABI}" \
  "${ENABLED_DECODERS[@]}"
```

Sau khi hoàn tất, các file `.a` sẽ nằm tại `${FFMPEG_MODULE_PATH}/jni/ffmpeg/android-libs/`.

## 5. Build bản AAR bằng Gradle

Để tạo file `.aar` sử dụng trong dự án Android:

1.  Cấu hình `local.properties` tại thư mục `media/`:
    ```properties
    sdk.dir=/Users/admin/Library/Android/sdk
    ndk.dir=/Users/admin/Library/Android/sdk/ndk/27.0.12077973
    ```

2.  Chạy lệnh build:
    ```bash
    cd "path"
    ./gradlew :lib-decoder-ffmpeg:assembleRelease
    ```

File AAR kết quả sẽ nằm tại:
`media/libraries/decoder_ffmpeg/buildout/outputs/aar/lib-decoder-ffmpeg-release.aar`

## 6. Kiểm tra 16KB Alignment

Để đảm bảo các file thư viện đã được căn chỉnh 16KB thành công, sử dụng lệnh sau:

```bash
# Kiểm tra file .so bên trong AAR hoặc thư mục build
readelf -l <đường_dẫn_file_libffmpegJNI.so> | grep -A 1 'LOAD'
```

**Kết quả đúng**: Cột `Align` cuối cùng của các phân đoạn `LOAD` phải hiển thị **`0x4000`** (tương đương 16384 bytes).

---
*Lưu ý: Đối với kiến trúc 64-bit (arm64, x86_64), lệnh `readelf` có thể xuống dòng thông tin, hãy dùng `grep -A 1` để thấy đủ giá trị Align.*
