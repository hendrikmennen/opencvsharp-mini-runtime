# Workflow Configuration Changes

## Summary
Updated all GitHub workflows to support only Ubuntu 24.04 and macOS (x64 and arm64) platforms, and configured OpenCV with video capture support.

## Changes Made

### 1. Platform Support
**Removed platforms:**
- Windows (x64, x86, arm64)
- CentOS 7 (x64, arm64)
- RHEL 8 (x64, arm64)
- Alpine Linux (x64, arm64)
- Android (x64, arm64)

**Supported platforms:**
- Ubuntu 24.04 (x64, arm64)
- macOS (x64, arm64)

### 2. OpenCV Configuration
Updated OpenCV build configuration in [opencv.yml](.github/workflows/opencv.yml):
- **Added video capture module:** `BUILD_LIST=core,imgproc,imgcodecs,videoio`
- **Enabled video backends:**
  - `WITH_GSTREAMER=ON` - GStreamer support
  - `WITH_FFMPEG=ON` - FFmpeg support
  - `WITH_V4L=ON` - Video4Linux support (Linux)
  - `WITH_AVFOUNDATION=ON` - AVFoundation support (macOS)
- **Installed required dependencies:**
  - Ubuntu: `libavcodec-dev`, `libavformat-dev`, `libswscale-dev`, `libgstreamer1.0-dev`, `libgstreamer-plugins-base1.0-dev`, `libv4l-dev`, `v4l-utils`
  - macOS: `ffmpeg` (via Homebrew)

### 3. Workflow Updates
All workflows updated:
- [opencv.yml](.github/workflows/opencv.yml) - OpenCV build
- [opencvsharp.yml](.github/workflows/opencvsharp.yml) - OpenCvSharp build
- [make-nuget.yml](.github/workflows/make-nuget.yml) - NuGet package creation
- [test-opencvsharp.yml](.github/workflows/test-opencvsharp.yml) - OpenCvSharp testing
- [push-nuget.yml](.github/workflows/push-nuget.yml) - NuGet package publishing

### 4. Simplifications
- Removed all Docker-based build steps (no longer needed without CentOS/RHEL/Alpine)
- Removed Windows-specific configuration
- Removed Android NDK configuration
- Simplified build scripts to native runners only
- Changed branch trigger from `feature/docker` to `main`

## Video Capture Usage
With these changes, OpenCV will support:
- Camera access via V4L2 on Linux
- Camera access via AVFoundation on macOS
- Video file reading/writing via FFmpeg
- Video streaming via GStreamer

Example usage in OpenCvSharp:
```csharp
using OpenCvSharp;

// Open default camera
using var capture = new VideoCapture(0);
if (!capture.IsOpened())
{
    Console.WriteLine("Camera not found");
    return;
}

// Capture frame
using var frame = new Mat();
capture.Read(frame);
```
