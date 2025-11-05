# Steps to reproduce
1. Install the latest Android Studio 2025.2.1, the Android SDK and the NDK
2. Make a new project with no activity.
3. Select language Java, minimum SDK 24, Kotlin DSL
4. In the project panel, use the shortcut alt + insert and add a CMakeLists.txt
5. Paste the following:
```
cmake_minimum_required(VERSION 3.17)

project(bug C)

add_library(bug SHARED jni/src/main.c)

target_compile_options(bug PUBLIC -fsanitize=address -fno-omit-frame-pointer)
set_target_properties(bug PROPERTIES LINK_FLAGS -fsanitize=address)

find_package(SDL3 REQUIRED CONFIG COMPONENTS SDL3-shared)
target_link_libraries(bug PRIVATE SDL3::SDL3)
```
6. Create a new file and the parent directories `app/jni/src/main.c` and paste the following:
```c
#include <SDL3/SDL.h>
#include <SDL3/SDL_main.h>

int main(int argc, char** argv) {
    return 0;
}
```
7. Copy the SDL `.aar` package contained in the [zip](https://github.com/libsdl-org/SDL/releases/download/release-3.2.26/SDL3-devel-3.2.26-android.zip) from GitHub releases to a new directory `app/libs`
8. In `app/build.gradle.kts`, under the top-level `android` section, add the following:
```kotlin
buildFeatures {
  prefab = true
}
externalNativeBuild {
  cmake {
    path("CMakeLists.txt")
  }
}
```
9. Add the following to the `dependencies` section: `implementation(files("libs/SDL3-3.2.26.aar"))`
10. Add a new Empty Views Activity. Don't generate a layout file, do make a launcher activity.
11. In the new activity change `extends AppCompatActivity` to `extends SDLActivity`
12. Add the following to the class body:
```java
@Override
protected String[] getLibraries() {
    return new String[] {
        "SDL3",
        "bug",
    };
}
```
13. Copy the file from the NDK `Android/Sdk/ndk/29.0.14206865/toolchains/llvm/prebuilt/linux-x86_64/lib/clang/21/lib/linux/libclang_rt.asan-aarch64-android.so` into a new directory `app/src/main/jniLibs/arm64-v8a`.
14. Run the app in debug mode. See the ASan errors in the logcat. Example output, note how the ASan output seems to be interleaved with other stuff:
```
11-05 00:27:03.413 23880 23880 V SDL     : Manufacturer: Xiaomi
11-05 00:27:03.413 23880 23880 V SDL     : Device: curtana
11-05 00:27:03.413 23880 23880 V SDL     : Model: Redmi Note 9S
11-05 00:27:03.413 23880 23880 V SDL     : onCreate()
11-05 00:27:03.551  1177  2522 E libnav  : AbaGetState return state 3 for feature 4
11-05 00:27:03.581  3086 10187 I SarService: Receiver polling >>>> Change receiver mode: off
11-05 00:27:03.621  1177  2522 E libnav  : AbaGetState return state 3 for feature 4
11-05 00:27:03.698 23880 23880 D nativeloader: Load /data/app/~~UA_YgitvkOEUmRjIBzve-Q==/me.elinge.sdlbug1-RgMZLe4SZaehMtiPOyozZA==/base.apk!/lib/arm64-v8a/libSDL3.so using class loader ns clns-4 (caller=/data/app/~~UA_YgitvkOEUmRjIBzve-Q==/me.elinge.sdlbug1-RgMZLe4SZaehMtiPOyozZA==/base.apk!classes4.dex): ok
11-05 00:27:03.730  1177  2522 E libnav  : AbaGetState return state 3 for feature 4
11-05 00:27:03.781  3086 10187 I SarService: Receiver polling >>>> Change receiver mode: off
11-05 00:27:03.910 23880 23880 D nativeloader: Load /data/app/~~UA_YgitvkOEUmRjIBzve-Q==/me.elinge.sdlbug1-RgMZLe4SZaehMtiPOyozZA==/base.apk!/lib/arm64-v8a/libbug.so using class loader ns clns-4 (caller=/data/app/~~UA_YgitvkOEUmRjIBzve-Q==/me.elinge.sdlbug1-RgMZLe4SZaehMtiPOyozZA==/base.apk!classes4.dex): ok
11-05 00:27:03.911  1177  2522 E libnav  : AbaGetState return state 3 for feature 4
11-05 00:27:03.911 23880 23880 V SDL     : nativeSetupJNI()
11-05 00:27:03.913 23880 23880 V SDL     : AUDIO nativeSetupJNI()
11-05 00:27:03.916 23880 23880 V SDL     : CONTROLLER nativeSetupJNI()
11-05 00:27:03.937 23880 24682 I me.elinge.sdlbug1: =================================================================
11-05 00:27:03.937 23880 24682 I me.elinge.sdlbug1: ==23880==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000000 (pc 0x0000680b0058 bp 0x00719bba83e0 sp 0x00719bba8370 T-1)
11-05 00:27:03.937 23880 24682 I me.elinge.sdlbug1: ==23880==The signal is caused by a READ memory access.
11-05 00:27:03.937 23880 24682 I me.elinge.sdlbug1: ==23880==Hint: address points to the zero page.
11-05 00:27:03.981  3086 10187 I SarService: Receiver polling >>>> Change receiver mode: off
11-05 00:27:04.036 23880 24682 I me.elinge.sdlbug1:     #0 0x0000680b0058  (/memfd:jit-cache (deleted)+0x20b0058)
11-05 00:27:04.036 23880 24682 I me.elinge.sdlbug1:     #1 0x00719aad7370  (/data/app/~~UA_YgitvkOEUmRjIBzve-Q==/me.elinge.sdlbug1-RgMZLe4SZaehMtiPOyozZA==/base.apk!/lib/arm64-v8a/libclang_rt.asan-aarch64-android.so+0x74370) (BuildId: 143dec4153d2acb2a3f10e5ff97e1c35acaadc38)
11-05 00:27:04.036 23880 24682 I me.elinge.sdlbug1:     #2 0x00719ab4eae8  (/data/app/~~UA_YgitvkOEUmRjIBzve-Q==/me.elinge.sdlbug1-RgMZLe4SZaehMtiPOyozZA==/base.apk!/lib/arm64-v8a/libclang_rt.asan-aarch64-android.so+0xebae8) (BuildId: 143dec4153d2acb2a3f10e5ff97e1c35acaadc38)
11-05 00:27:04.036 23880 24682 I me.elinge.sdlbug1:     #3 0x00719ab4e8cc  (/data/app/~~UA_YgitvkOEUmRjIBzve-Q==/me.elinge.sdlbug1-RgMZLe4SZaehMtiPOyozZA==/base.apk!/lib/arm64-v8a/libclang_rt.asan-aarch64-android.so+0xeb8cc) (BuildId: 143dec4153d2acb2a3f10e5ff97e1c35acaadc38)
11-05 00:27:04.036 23880 24682 I me.elinge.sdlbug1:     #4 0x00719ab4dd54  (/data/app/~~UA_YgitvkOEUmRjIBzve-Q==/me.elinge.sdlbug1-RgMZLe4SZaehMtiPOyozZA==/base.apk!/lib/arm64-v8a/libclang_rt.asan-aarch64-android.so+0xead54) (BuildId: 143dec4153d2acb2a3f10e5ff97e1c35acaadc38)
11-05 00:27:04.036 23880 24682 I me.elinge.sdlbug1:     #5 0x0072cbb90664  ([vdso]+0x664) (BuildId: 829006729089a3b740a18b084d533ddf5f06df6b)
11-05 00:27:04.036 23880 24682 I me.elinge.sdlbug1: 
11-05 00:27:04.036  1601  2021 W ActivityManager: Scheduling restart of crashed service com.facebook.services/com.facebook.oxygen.services.fbns.PreloadedFbnsService in 1000ms for start-requested
11-05 00:27:04.036 23880 24682 I me.elinge.sdlbug1: AddressSanitizer can not provide additional info.
11-05 00:27:04.078 23880 24682 I me.elinge.sdlbug1: SUMMARY: AddressSanitizer: SEGV (/memfd:jit-cache (deleted)+0x20b0058) 
11-05 00:27:04.078 23880 24682 I me.elinge.sdlbug1: ==23880==ABORTING
```
