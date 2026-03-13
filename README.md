# gst-plugins-qti-oss

Qualcomm Technologies, Inc. open-source GStreamer components for Linux-based Qualcomm platforms.

This repository contains:

- platform utility libraries used by GStreamer pipelines
- hardware-accelerated and ML-focused GStreamer plugins
- command-line tools and sample applications
- test utilities and a UVC camera daemon
- container reference flows for reproducible runtime/build environments

## Repository layout

- gst-ml-metadata: shared ML metadata definitions used across plugins
- gst-plugin-base: common utility libraries (video, memory, allocators, ML, CV helpers)
- gst-plugin-\*: plugin modules (camera, codec, ML, CV, overlay, transform, tracking, networking, broker, etc.)
- gst-plugin-tools: utility binaries (pipeline app, rtsp server app, warmboot app, ML apps)
- gst-plugin-examples: gated example applications, enabled by platform feature flags
- gst-sample-apps: standalone sample applications for camera, playback, encode/decode, AI, and streaming
- gst-test-framework: GStreamer-based test binary
- gst-umd-daemon: UVC camera daemon utility
- gst-python-examples: Python-based examples
- gst-docker-ref: layered Docker reference images and setup guidance

## Build system model

This repo is organized as many CMake subprojects (one per plugin/app/tool) instead of one top-level CMake entry.

Most CMake files expect platform-provided cache variables/toolchain settings (commonly injected by an external build system), including values such as:

- GST_VERSION_REQUIRED
- GST_PLUGINS_QTI_OSS_VERSION
- SYSROOT_INCDIR
- SYSROOT_LIBDIR
- GST_PLUGINS_QTI_OSS_INSTALL_BINDIR
- GST_PLUGINS_QTI_OSS_INSTALL_LIBDIR
- GST_PLUGINS_QTI_OSS_INSTALL_CONFIG
- KERNEL_BUILDDIR (for modules that probe kernel headers)

Some projects are conditionally compiled by feature toggles such as:

- ENABLE_CAMERA
- ENABLE_DISPLAY
- ENABLE_VIDEO_ENCODE
- ENABLE_VIDEO_DECODE
- ENABLE_ML
- ENABLE_WEBRTC

## Prerequisites

At minimum, your environment should provide:

- CMake 3.8+ (some projects request 3.16)
- pkg-config
- GStreamer 1.x development packages (core/base/video/audio/check/rtsp-server as required)
- a valid sysroot with Qualcomm platform headers and libraries for target-specific components
- kernel headers when building modules that check kernel media interfaces

Note:

- Several plugins depend on proprietary or platform-specific SDK/driver headers and will not build in a generic desktop environment without the target sysroot.

## Building a component

Because each directory is a standalone CMake project, build the component you need directly.

Example pattern:

1. Configure:

   cmake -S gst-plugin-batch -B build/gst-plugin-batch \
    -DGST_VERSION_REQUIRED=1.20 \
    -DGST_PLUGINS_QTI_OSS_VERSION=1.0.0 \
    -DSYSROOT_INCDIR=<path-to-sysroot-include> \
    -DSYSROOT_LIBDIR=<path-to-sysroot-lib> \
    -DGST_PLUGINS_QTI_OSS_INSTALL_LIBDIR=<install-lib-dir> \
    -DGST_PLUGINS_QTI_OSS_INSTALL_BINDIR=<install-bin-dir> \
    -DGST_PLUGINS_QTI_OSS_INSTALL_CONFIG=<install-config-dir>

2. Build:

   cmake --build build/gst-plugin-batch -j

3. Install:

   cmake --install build/gst-plugin-batch

Repeat the same pattern for other directories, for example:

- gst-plugin-base
- gst-plugin-codec2
- gst-plugin-mlqnn
- gst-plugin-objtracker
- gst-plugin-tools
- gst-test-framework

## Runtime validation

After installation, ensure GStreamer can discover the plugins:

1. Set plugin search path if needed:

   export GST_PLUGIN_PATH=<install-lib-dir>/gstreamer-1.0:$GST_PLUGIN_PATH

2. Inspect available QTI plugins:

   gst-inspect-1.0 | grep -i qti

3. Inspect a specific plugin:

   gst-inspect-1.0 gstqtibatch

4. Run an application from gst-sample-apps or gst-plugin-examples that matches your enabled features.

## Testing

Build and run the test utility in gst-test-framework when your target environment supports the required camera/media stack.

## Docker reference workflow

See gst-docker-ref for layered reference images:

- runtimedepsimage
- mltoolsimage
- gstcoreimage
- qimpluginsimage
- finalappimage

Start with:

- gst-docker-ref/README.md

Then follow each subdirectory README for image-specific steps.

## Notes for integrators

- Prefer integrating these CMake subprojects from your platform build system so common variables/toolchain paths are injected consistently.
- Build failures are often due to missing target headers/libs in the selected sysroot rather than source-level issues.
- Feature-gated examples only appear when matching ENABLE\_\* options and dependent headers are available.
