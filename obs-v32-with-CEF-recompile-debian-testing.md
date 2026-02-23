---
layout: default
title: "OBS v32 with CEF support recompile on debian testing (Feb 2026)"
permalink: /obs-v32-with-CEF-recompile-debian-testing/
---
## Guide: Compiling OBS Studio 32 on Debian Testing with Browser Source Enabled (February 2026)

This guide outlines the tested and working steps to compile OBS Studio (version 32) from source code on Debian Testing, including support for the "Browser Source" via CEF (Chromium Embedded Framework). 

Compiling from source often gets stuck due to missing dependencies, incompatible C++ versions, or wrong paths. This procedure bypasses the known issues and gets straight to the solution.

### 1. Installing Preliminary Dependencies

To avoid most `cmake` errors caused by missing packages, we use the official Debian repositories to download all basic build dependencies.

Open the terminal and run:
```bash
sudo apt update
sudo apt install build-essential cmake cpack checkinstall extra-cmake-modules libsimde-dev
sudo apt build-dep obs-studio
```

### 2. Downloading and Preparing CEF (CRITICAL Step)

**Warning:** Do not download the very latest version of CEF/Chromium (e.g., version 145)! Recent versions require the C++20 standard, which conflicts with current OBS plugins. You must use build **5060**, which is recommended by the OBS team for Linux.

Let's download and extract the archive:
```bash
wget https://cdn-fastly.obsproject.com/downloads/cef_binary_5060_linux64.tar.bz2
tar -xvf cef_binary_5060_linux64.tar.bz2
cd cef_binary_5060_linux64
```

Now we need to **compile the CEF C++ wrapper**, otherwise CMake won't be able to find the `CEF_LIBRARY_WRAPPER_RELEASE` file:
```bash
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j$(nproc) libcef_dll_wrapper
```

### 3. Configuring OBS with CMake

Navigate to the main folder where you cloned the OBS Studio source code and create a clean `build` directory:
```bash
cd /path/to/your/obs-studio/
mkdir build && cd build
```

Now let's run `cmake`. 
*Note: Replace `/absolute/path/to/...` with the actual path where you extracted the `cef_binary_5060_linux64` folder in the previous step.*

```bash
cmake -DENABLE_BROWSER=ON \
      -DCEF_ROOT_DIR=/absolute/path/to/cef_binary_5060_linux64/ \
      -DCEF_INCLUDE_DIR=/absolute/path/to/cef_binary_5060_linux64/include \
      -DENABLE_AJA=OFF \
      -DENABLE_WEBRTC=OFF \
      -DCMAKE_INSTALL_PREFIX=/usr \
      ..
```

**Explanation of the flags:**
* `-DENABLE_BROWSER=ON`: Enables the Browser module (in older versions, this was `BUILD_BROWSER`).
* `-DENABLE_AJA=OFF` and `-DENABLE_WEBRTC=OFF`: Disables plugins for professional capture hardware and WebRTC, avoiding the need for non-standard external libraries.
* `-DCMAKE_INSTALL_PREFIX=/usr`: Crucial on Debian to install the libraries in standard system paths and avoid the `libobs-frontend-api.so.30: cannot open shared object file` error upon launch.

#### CMake Result:
If everything went smoothly, you should see `obs-browser` listed among the enabled modules.

```text
CMake Warning (dev) at cmake/common/versionconfig.cmake:59 (message):
  
  ******************************************************************************

    + OBS-Studio - Release candidate detected, OBS_VERSION is now: 32.1.0-rc1-20-gacd4c9787

  
  ******************************************************************************
Call Stack (most recent call first):
  cmake/common/bootstrap.cmake:60 (include)
  CMakeLists.txt:3 (include)
This warning is for project developers.  Use -Wno-dev to suppress it.

-- The C compiler identification is GNU 15.2.0
-- The CXX compiler identification is GNU 15.2.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Checking for interprocedural optimization support
-- Checking for interprocedural optimization support - available
-- Checking for interprocedural optimization support - enabled [Release, MinSizeRel]
-- Found SIMDe: /usr/include (found version "0.8.2")
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD - Success
-- Found Threads: TRUE
-- Found FFmpeg: /usr/lib/x86_64-linux-gnu/libavformat.so;/usr/lib/x86_64-linux-gnu/libavutil.so;/usr/lib/x86_64-linux-gnu/libswscale.so;/usr/lib/x86_64-linux-gnu/libswresample.so;/usr/lib/x86_64-linux-gnu/libavcodec.so (found suitable version "8.0", minimum required is "6.1") found components: avformat avutil swscale swresample avcodec
-- Found ZLIB: /usr/lib/x86_64-linux-gnu/libz.so (found version "1.3.1")
-- Found Uthash: /usr/include (found version "2.3.0")
-- Found jansson: /usr/lib/x86_64-linux-gnu/libjansson.so (found version "2.14")
-- Found LibUUID: /usr/lib/x86_64-linux-gnu/libuuid.so (found version "2.41.3")
-- Found X11: /usr/include
-- Looking for XOpenDisplay in /usr/lib/x86_64-linux-gnu/libX11.so;/usr/lib/x86_64-linux-gnu/libXext.so
-- Looking for XOpenDisplay in /usr/lib/x86_64-linux-gnu/libX11.so;/usr/lib/x86_64-linux-gnu/libXext.so - found
-- Looking for gethostbyname
-- Looking for gethostbyname - found
-- Looking for connect
-- Looking for connect - found
-- Looking for remove
-- Looking for remove - found
-- Looking for shmat
-- Looking for shmat - found
-- Looking for IceConnectionNumber in ICE
-- Looking for IceConnectionNumber in ICE - found
-- Found X11_XCB: /usr/lib/x86_64-linux-gnu/libX11-xcb.so (found version "1.8.12")
-- Found XCB_XCB: /usr/lib/x86_64-linux-gnu/libxcb.so (found version "1.17.0")
-- Found XCB_XINPUT: /usr/lib/x86_64-linux-gnu/libxcb-xinput.so (found version "1.17.0")
-- Found XCB: /usr/lib/x86_64-linux-gnu/libxcb.so;/usr/lib/x86_64-linux-gnu/libxcb-xinput.so (found version "1.17.0") found components: XCB XINPUT
-- Found Gio: /usr/lib/x86_64-linux-gnu/libgio-2.0.so (found version "2.87.2")
-- Performing Test HAVE_MATH_IN_STD_LIB
-- Performing Test HAVE_MATH_IN_STD_LIB - Failed
-- Performing Test HAVE_UUID_HEADER
-- Performing Test HAVE_UUID_HEADER - Success
-- Found PulseAudio: /usr/include (found version "17.0")
-- Found Wayland_Client: /usr/lib/x86_64-linux-gnu/libwayland-client.so (found version "1.24.0")
-- Found Wayland: /usr/lib/x86_64-linux-gnu/libwayland-client.so (found version "1.24.0") found components: Client
-- Found Xkbcommon: /usr/lib/x86_64-linux-gnu/libxkbcommon.so (found version "1.13.1")
-- Performing Test COMPILER_HAS_HIDDEN_VISIBILITY
-- Performing Test COMPILER_HAS_HIDDEN_VISIBILITY - Success
-- Performing Test COMPILER_HAS_HIDDEN_INLINE_VISIBILITY
-- Performing Test COMPILER_HAS_HIDDEN_INLINE_VISIBILITY - Success
-- Performing Test COMPILER_HAS_DEPRECATED_ATTR
-- Performing Test COMPILER_HAS_DEPRECATED_ATTR - Success
-- Found OpenGL: /usr/lib/x86_64-linux-gnu/libOpenGL.so
-- Found Libdrm: /usr/lib/x86_64-linux-gnu/libdrm.so (found version "2.4.131")
-- Found XCB: /usr/lib/x86_64-linux-gnu/libxcb.so (found version "1.17.0") found components: XCB
-- Found OpenGL: /usr/lib/x86_64-linux-gnu/libOpenGL.so  found components: EGL
-- Found Wayland_Server: /usr/lib/x86_64-linux-gnu/libwayland-server.so (found version "1.24.0")
-- Found Wayland_Cursor: /usr/lib/x86_64-linux-gnu/libwayland-cursor.so (found version "1.24.0")
-- Found Wayland_Egl: /usr/lib/x86_64-linux-gnu/libwayland-egl.so (found version "18.1.0")
-- Found Wayland: /usr/lib/x86_64-linux-gnu/libwayland-client.so;/usr/lib/x86_64-linux-gnu/libwayland-server.so;/usr/lib/x86_64-linux-gnu/libwayland-cursor.so;/usr/lib/x86_64-linux-gnu/libwayland-egl.so (found version "1.24.0")
-- Found ALSA: /usr/lib/x86_64-linux-gnu/libasound.so (found version "1.2.15.3")
-- XCB: XFIXES requires XCB;RENDER;SHAPE
-- XCB: XFIXES requires XCB;RENDER;SHAPE
-- Found XCB_RENDER: /usr/lib/x86_64-linux-gnu/libxcb-render.so (found version "1.17.0")
-- Found XCB_SHAPE: /usr/lib/x86_64-linux-gnu/libxcb-shape.so (found version "1.17.0")
-- Found XCB_XFIXES: /usr/lib/x86_64-linux-gnu/libxcb-xfixes.so (found version "1.17.0")
-- Found XCB_SHM: /usr/lib/x86_64-linux-gnu/libxcb-shm.so (found version "1.17.0")
-- Found XCB_COMPOSITE: /usr/lib/x86_64-linux-gnu/libxcb-composite.so (found version "1.17.0")
-- Found XCB_RANDR: /usr/lib/x86_64-linux-gnu/libxcb-randr.so (found version "1.17.0")
-- Found XCB_XINERAMA: /usr/lib/x86_64-linux-gnu/libxcb-xinerama.so (found version "1.17.0")
-- Found XCB: /usr/lib/x86_64-linux-gnu/libxcb.so;/usr/lib/x86_64-linux-gnu/libxcb-render.so;/usr/lib/x86_64-linux-gnu/libxcb-shape.so;/usr/lib/x86_64-linux-gnu/libxcb-xfixes.so;/usr/lib/x86_64-linux-gnu/libxcb-shm.so;/usr/lib/x86_64-linux-gnu/libxcb-composite.so;/usr/lib/x86_64-linux-gnu/libxcb-randr.so;/usr/lib/x86_64-linux-gnu/libxcb-xinerama.so (found version "1.17.0") found components: XCB XFIXES RANDR SHM XINERAMA COMPOSITE
-- Found PipeWire: /usr/lib/x86_64-linux-gnu/libpipewire-0.3.so (found suitable version "1.4.10", minimum required is "0.3.33")
-- Found Gio: /usr/lib/x86_64-linux-gnu/libgio-2.0.so (found suitable version "2.87.2", minimum required is "2.76")
-- Found Libv4l2: /usr/lib/x86_64-linux-gnu/libv4l2.so (found version "1.32.0")
-- Found FFmpeg: /usr/lib/x86_64-linux-gnu/libavcodec.so;/usr/lib/x86_64-linux-gnu/libavutil.so;/usr/lib/x86_64-linux-gnu/libavformat.so (found version "8.0") found components: avcodec avutil avformat
-- Performing Test HAVE_VIDEODEV2_HEADER
-- Performing Test HAVE_VIDEODEV2_HEADER - Success
-- Found Libudev: /usr/lib/x86_64-linux-gnu/libudev.so (found version "259")
-- Found CEF: /absolute/path/to/cef_binary_5060_linux64/Release/libcef.so (found suitable version "103.0.0", minimum required is "95")
-- Found nlohmann_json: /usr/share/cmake/nlohmann_json/nlohmann_jsonConfig.cmake (found suitable version "3.11.3", minimum required is "3.11")
-- Performing Test HAVE_STDATOMIC
-- Performing Test HAVE_STDATOMIC - Success
-- Found WrapAtomic: TRUE
-- Found OpenGL: /usr/lib/x86_64-linux-gnu/libOpenGL.so
-- Found WrapOpenGL: TRUE
-- Found WrapVulkanHeaders: /usr/include
-- Found FFmpeg: /usr/lib/x86_64-linux-gnu/libavcodec.so;/usr/lib/x86_64-linux-gnu/libavfilter.so;/usr/lib/x86_64-linux-gnu/libavdevice.so;/usr/lib/x86_64-linux-gnu/libavutil.so;/usr/lib/x86_64-linux-gnu/libswscale.so;/usr/lib/x86_64-linux-gnu/libavformat.so;/usr/lib/x86_64-linux-gnu/libswresample.so (found suitable version "8.0", minimum required is "6.1") found components: avcodec avfilter avdevice avutil swscale avformat swresample
-- Found FFmpeg: /usr/lib/x86_64-linux-gnu/libavcodec.so;/usr/lib/x86_64-linux-gnu/libavfilter.so;/usr/lib/x86_64-linux-gnu/libavdevice.so;/usr/lib/x86_64-linux-gnu/libavutil.so;/usr/lib/x86_64-linux-gnu/libswscale.so;/usr/lib/x86_64-linux-gnu/libavformat.so;/usr/lib/x86_64-linux-gnu/libswresample.so (found version "8.0") found components: avcodec avdevice avutil avformat
-- Found Libva: /usr/lib/x86_64-linux-gnu/libva.so (found version "1.23.0")
-- Found Libpci: /usr/lib/x86_64-linux-gnu/libpci.so (found version "3.14.0")
-- Found FFmpeg: /usr/lib/x86_64-linux-gnu/libavcodec.so;/usr/lib/x86_64-linux-gnu/libavfilter.so;/usr/lib/x86_64-linux-gnu/libavdevice.so;/usr/lib/x86_64-linux-gnu/libavutil.so;/usr/lib/x86_64-linux-gnu/libswscale.so;/usr/lib/x86_64-linux-gnu/libavformat.so;/usr/lib/x86_64-linux-gnu/libswresample.so (found version "8.0") found components: avcodec avutil avformat
-- Found Libspeexdsp: /usr/lib/x86_64-linux-gnu/libspeexdsp.so (found version "1.2.1")
CMake Warning (dev) at cmake/finders/FindLibrnnoise.cmake:87 (message):
  Failed to find Librnnoise version.
Call Stack (most recent call first):
  plugins/obs-filters/cmake/rnnoise.cmake:7 (find_package)
  plugins/obs-filters/CMakeLists.txt:35 (include)
This warning is for project developers.  Use -Wno-dev to suppress it.

-- Could NOT find Librnnoise (missing: Librnnoise_LIBRARY Librnnoise_INCLUDE_DIR) (found version "0.0.0")
    Reason given by package: Ensure librnnoise libraries are available in local libary paths.

CMake Warning at plugins/obs-filters/cmake/rnnoise.cmake:10 (message):
  No RNNoise library found.  Using internal RNNoise version instead.
Call Stack (most recent call first):
  plugins/obs-filters/CMakeLists.txt:35 (include)


-- Found FFnvcodec: /usr/include (found suitable version "12.1.14.0", minimum required is "12")
-- Found VPL: /usr/lib/x86_64-linux-gnu/libvpl.so (Required is at least version "2.9")
-- Found qrcodegencpp: /usr/lib/x86_64-linux-gnu/libqrcodegencpp.so (found version "1.8.0")
-- Found Websocketpp: /usr/include (found suitable version "0.8.3", minimum required is "0.8")
-- Found Asio: /usr/include (found suitable version "1.30.2", minimum required is "1.12.1")
-- Found Libx264: /usr/lib/x86_64-linux-gnu/libx264.so (found version "0.165.x")
-- Found CURL: /usr/lib/x86_64-linux-gnu/libcurl.so (found version "8.18.0")
-- Found Freetype: /usr/lib/x86_64-linux-gnu/libfreetype.so (found version "2.14.1")
-- Found Fontconfig: /usr/lib/x86_64-linux-gnu/libfontconfig.so (found version "2.17.1")
-- Found FFmpeg: /usr/lib/x86_64-linux-gnu/libavcodec.so;/usr/lib/x86_64-linux-gnu/libavutil.so;/usr/lib/x86_64-linux-gnu/libavformat.so (found version "8.0") found components: avcodec avutil avformat
-- Found SWIG: /usr/bin/swig (found suitable version "4.4.0", minimum required is "4")
-- Found Luajit: /usr/lib/x86_64-linux-gnu/libluajit-5.1.so (found version "2.1.1761786044")
CMake Warning (dev) at shared/obs-scripting/cmake/lua.cmake:8 (add_custom_command):
  The following keywords are not supported when using
  add_custom_command(OUTPUT): PRE_BUILD.

  Policy CMP0175 is not set: add_custom_command() rejects invalid arguments.
  Run "cmake --help-policy CMP0175" for policy details.  Use the cmake_policy
  command to set the policy and suppress this warning.
Call Stack (most recent call first):
  shared/obs-scripting/CMakeLists.txt:15 (include)
This warning is for project developers.  Use -Wno-dev to suppress it.

-- Found Python: /usr/bin/python3 (found suitable version "3.13.12", minimum required is "3.8") found components: Interpreter Development Development.Module Development.Embed
CMake Warning (dev) at shared/obs-scripting/cmake/python.cmake:14 (add_custom_command):
  The following keywords are not supported when using
  add_custom_command(OUTPUT): PRE_BUILD.

  Policy CMP0175 is not set: add_custom_command() rejects invalid arguments.
  Run "cmake --help-policy CMP0175" for policy details.  Use the cmake_policy
  command to set the policy and suppress this warning.
Call Stack (most recent call first):
  shared/obs-scripting/CMakeLists.txt:16 (include)
This warning is for project developers.  Use -Wno-dev to suppress it.

-- Found Python: /usr/bin/python3 (found version "3.13.12") found components: Interpreter Development Development.Module Development.Embed
                      _                   _             _ _       
                 ___ | |__  ___       ___| |_ _   _  __| (_) ___  
                / _ \| '_ \/ __|_____/ __| __| | | |/ _` | |/ _ \ 
               | (_) | |_) \__ \_____\__ \ |_| |_| | (_| | | (_) |
                \___/|_.__/|___/     |___/\__|\__,_|\__,_|_|\___/ 

OBS:  Application Version: 32.1.0-rc1-20-gacd4c9787 - Build Number: 17
==================================================================================


------------------------       Enabled Features           ------------------------
 - Browser panels
 - OpenGL renderer
 - PipeWire 0.3.60+ camera support
 - Plugin Support
 - PulseAudio audio monitoring (Linux)
 - RNNoise noise suppression
 - Scripting Support (Frontend)
 - Scripting support
 - SpeexDSP noise suppression
 - User Interface
 - Wayland compositor support (Linux)
 - What's New panel
------------------------       Disabled Features          ------------------------
 - Idian Playground
 - Restream API connection
 - Twitch API connection
 - YouTube API connection
------------------------        Enabled Modules           ------------------------
 - decklink
 - decklink-captions
 - decklink-output-ui
 - frontend-tools
 - image-source
 - linux-alsa
 - linux-capture
 - linux-pipewire
 - linux-pulseaudio
 - linux-v4l2
 - obs-browser
 - obs-ffmpeg
 - obs-filters
 - obs-nvenc
 - obs-outputs
 - obs-qsv11
 - obs-transitions
 - obs-vst
 - obs-websocket
 - obs-x264
 - obslua
 - obspython
 - rtmp-services
 - text-freetype2
 - vlc-video
------------------------        Disabled Modules          ------------------------
 - aja
 - aja-output-ui
 - linux-jack
 - obs-libfdk
 - obs-webrtc
 - sndio
 - test-input
----------------------------------------------------------------------------------
-- Configuring done (6.1s)
-- Generating done (0.2s)
-- Build files have been written to: /path/to/your/obs-studio/build
```

### 4. Compiling and Creating the .deb Package

Let's start the actual compilation, utilizing all available CPU cores to speed things up:
```bash
make -j$(nproc)
```

```text
[...]
[100%] Linking CXX executable obs
Copy obs-studio to binary directory
Copy obs-studio resource /path/to/your/obs-studio/frontend/../AUTHORS to library directory (share/obs/obs-studio/authors)
Copy obs-studio resources to data directory (share/obs/obs-studio)
[100%] Built target obs-studio
```

Once it reaches 100%, we'll create the `.deb` installation package using CPack:
```bash
cpack -G DEB
```

```text
CPack: Create package using DEB
CPack: Install projects
CPack: - Run preinstall target for: obs-studio
CPack: - Install project: obs-studio []
CPack: Create package
CPackDeb: - Generating dependency list
CPack: - package: /path/to/your/obs-studio/build/obs-studio-32.1.0-rc1-20-gacd4c9787-Linux.deb generated.
CPack: - checksum file: /path/to/your/obs-studio/build/obs-studio-32.1.0-rc1-20-gacd4c9787-Linux.deb.sha256 generated.
CPack: - package: /path/to/your/obs-studio/build/obs-studio-32.1.0-rc1-20-gacd4c9787-Linux-dbgsym.ddeb generated.
CPack: - checksum file: /path/to/your/obs-studio/build/obs-studio-32.1.0-rc1-20-gacd4c9787-Linux-dbgsym.ddeb.sha256 generated.
```

### 5. System Cleanup and Installation

Before installing our custom build, it is **highly recommended** to remove any old OBS versions or leftover system libraries to avoid conflicts (like the dreaded `symbol lookup error`):

```bash
sudo apt remove --purge obs-studio libobs-dev libobs0*
sudo ldconfig
```

Finally, install the newly created package:
```bash
sudo apt install ./obs-studio-*.deb
```

### 6. Launch and Test

Launch OBS Studio. Go to the **Sources** dock, click the **+** button, and look for the **Browser** option. Add it, enter a URL, and verify that the preview loads the webpage correctly. 

If you see the website rendered in the canvas, your setup was successful! By enabling the Browser source, you will now also be able to use the integrated API logins for Twitch, YouTube, and Restream.