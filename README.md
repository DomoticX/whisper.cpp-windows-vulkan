![whisper.cpp Vulkan support](whisper.cpp_vulkan_banner.jpg)

![Windows](https://img.shields.io/badge/Platform-Windows-blue)
![Vulkan](https://img.shields.io/badge/GPU-Vulkan-orange)
![AMD](https://img.shields.io/badge/GPU-AMD-red)

# whisper.cpp with VULKAN (AMD GPU) on Windows

## Works on

- Vulkan supported videocards
- Windows 10/11 x64

By default, **whisper.cpp** builds typically run on CPU or CUDA.

If you have an **AMD GPU**, you can accelerate inference using **Vulkan**, but this requires compiling whisper.cpp from source with Vulkan support enabled.

This guide explains how to build **whisper.cpp with Vulkan support on Windows**.

---

# Requirements

You will need the following software installed:

- Vulkan SDK
- CMake
- Git
- Visual Studio Build Tools (usually already present on most developer systems)

---

# 1. Install Vulkan SDK

Download the Vulkan SDK from: https://vulkan.lunarg.com/sdk/home

Choose: Windows x64 Installer
Example: vulkansdk-windows-X64-1.x.xxx.x.exe

During installation:

- Select **Install Everything**
- Allow the installer to **set environment variables**

After installation open a **Windows Command Prompt** and run:
```
echo %VULKAN_SDK%
```

You should see something similar to:
```
C:\VulkanSDK\1.3.xxx.x
```

If this path appears, Vulkan is correctly installed.

---

# 2. Install CMake

Open a **Windows Command Prompt** and run:
```
winget install Kitware.CMake
```

Example output:
```
Found CMake [Kitware.CMake] Version 4.2.3

This application is licensed to you by its owner.
Microsoft is not responsible for, nor does it grant any licenses to third-party packages.

Downloading https://github.com/Kitware/CMake/releases/download/v4.2.3/cmake-4.2.3-windows-x86_64.msi

██████████████████████████████ 34.7 MB / 34.7 MB

Successfully verified installer hash
Starting package install...
Successfully installed
```

---

# 3. Clone whisper.cpp source

Open a **Windows Command Prompt** and run:
```
git clone https://github.com/ggml-org/whisper.cpp.git

Example output:
```
Cloning into 'whisper.cpp'...

remote: Enumerating objects: 31665, done.
remote: Counting objects: 100% (149/149), done.
remote: Compressing objects: 100% (75/75), done.
remote: Total 31665 (delta 79), reused 74 (delta 74), pack-reused 31516

Receiving objects: 100% (31665/31665), 33.63 MiB | 23.60 MiB/s, done.
Resolving deltas: 100% (23050/23050), done.
```

---

# 4. Build whisper.cpp with Vulkan support

Navigate to the source folder:
```
cd whisper.cpp
```
Optional: remove previous build directory for a clean build:
```
rmdir /s /q build
```

---

## Configure CMake with Vulkan

Enable Vulkan using:
```
cmake -B build -DGGML_VULKAN=1
```

---

## Important: AVX512 compatibility

Some CPUs **crash when AVX512 is enabled**.

| CPU | AVX512 |
|-----|-------|
| Intel 12th/13th gen | ❌ |
| AMD Ryzen | ❌ |
| Xeon / Threadripper | ⚠️ sometimes |

For maximum compatibility disable AVX512:

```
cmake -B build -DGGML_VULKAN=1 -DGGML_AVX512=OFF
```

---

## SIMD features still enabled

Even with AVX512 disabled **ggml still uses optimized CPU instructions**:

- AVX2
- FMA
- OpenMP

You will see these enabled in the build log, so CPU performance remains good.

---

# 5. Compile the binaries

Once configuration is complete build the project:
```
cmake --build build --config Release
```

After the build finishes the binaries will be located in:
```
build\bin\Release\
```

---

# Generated binaries

Example files produced by the build:
```
whisper-vad-speech-segments.exe
whisper-server.exe
whisper-quantize.exe
whisper-cli.exe
whisper-bench.exe

test-vad-full.exe
test-vad.exe
main.exe
bench.exe

whisper.dll
ggml.dll
ggml-vulkan.dll
ggml-cpu.dll
ggml-base.dll
```

---

# Where did `whisper-stream.exe` go?

Older versions of whisper.cpp included:

```
examples/stream/
```

This example used **PortAudio or SDL** for microphone capture.

It is often **not built by default anymore** because:

- Audio capture APIs differ per operating system
- Many users prefer using **FFmpeg pipelines** for streaming audio input

---

# Notes

Vulkan acceleration can significantly improve performance on **AMD GPUs** compared to CPU-only inference.

Performance depends on:

- GPU model
- Vulkan driver quality
- Whisper model size (tiny → large)

---

# Useful commands

Benchmark the system:
```
whisper-bench.exe
```

Run transcription:
```
whisper-cli.exe -m models/ggml-large-v3.bin -f audio.wav
```

---

# Links

Official repository:
https://github.com/ggml-org/whisper.cpp

Other Vulkan builds:
https://github.com/jerryshell/whisper.cpp-windows-vulkan-bin
