# GStreamer and NNStreamer Installation Guide

## Overview

This guide explains how to:

1. Build **GStreamer** from source.
2. Build **NNStreamer** from source using the custom-built GStreamer.
3. Configure the environment for development and testing.
4. Verify the installation.

---

# Part 1: Building GStreamer from Source

## Prerequisites

Install all required build dependencies before proceeding.

## 1. Clone GStreamer

```bash
git clone https://gitlab.freedesktop.org/gstreamer/gstreamer.git
cd gstreamer
```

## 2. Checkout Desired Branch

Example:

```bash
git checkout 1.24
```

## 3. Build and Compile

```bash
meson setup build
ninja -C build
```

## 4. Install (Optional)

By default, GStreamer is installed under `/usr/local`.

```bash
sudo ninja -C build install
```

> **Note:** Installation is optional. You can use the build directly through the GStreamer development environment.

## 5. Start GStreamer Environment

```bash
./gst-env.py
```

## 6. Verify Installation

```bash
gst-inspect-1.0 --version
```

Expected output should display the installed GStreamer version.

---

# Part 2: Building NNStreamer from Source

## Prerequisites

Before building NNStreamer:

* GStreamer must be built successfully.
* Either:

  * Install GStreamer into the system, or
  * Run NNStreamer inside the GStreamer build environment (`gst-env.py`).

---

## Install Required Dependencies

### NNStreamer Framework Dependencies

```bash
sudo add-apt-repository ppa:nnstreamer/ppa
sudo apt-get update

sudo apt-get install \
    libedgetpu-dev \
    libflatbuffers-dev \
    libgrpc-dev \
    openvino-dev \
    libpaho-mqtt-dev \
    libprotobuf-dev \
    pytorch \
    libopencv-dev \
    tensorflow2-lite-dev \
    tvm-runtime-dev
```

### Build Dependencies

```bash
sudo apt install \
    libglib2.0-dev \
    libjson-glib-dev \
    python3-dev \
    python3-numpy \
    flex \
    bison
```

---

## 1. Clone NNStreamer

```bash
git clone https://gitlab.com/nnstreamer/nnstreamer.git
cd nnstreamer
```

## 2. Checkout Desired Branch

Example:

```bash
git checkout 1.14
```

## 3. Build and Compile

```bash
meson setup build
meson compile -C build
```

## 4. Install (Optional)

```bash
sudo ninja -C build install
```

> **Note:** Installation is optional when using the build tree directly.

---

## 5. Add NNStreamer to GST_PLUGIN_PATH

```bash
export GST_PLUGIN_PATH="${GST_PLUGIN_PATH:+$GST_PLUGIN_PATH:}$HOME/codes/nnstreamer/build/gst/nnstreamer"
```

Replace:

```text
$HOME/codes/nnstreamer
```

with the actual location where NNStreamer was cloned.

---

## 6. Verify Tensor Filter Subplugins

Check that framework-specific libraries were generated:

```bash
find build/ext/nnstreamer/tensor_filter -name "*.so"
```

Expected examples:

```text
libnnstreamer_filter_tensorflow2-lite.so
libnnstreamer_filter_tvm.so
```

If present, TensorFlow Lite and TVM support were successfully built.

---

## 7. Configure NNStreamer

NNStreamer generates a configuration file:

```text
build/nnstreamer.ini
```

Export its location:

```bash
export NNSTREAMER_CONF=$HOME/codes/nnstreamer/build/nnstreamer.ini
```

### Why is this required?

Without the configuration file, errors such as:

```text
Failed to load configuration, no config file found
```

may occur, preventing framework subplugins from being discovered.

---

## 8. Configure Subplugin Paths

### Tensor Filter Plugins

```bash
export NNSTREAMER_FILTERS=$HOME/codes/nnstreamer/build/ext/nnstreamer/tensor_filter
```

### Tensor Decoder Plugins

```bash
export NNSTREAMER_DECODERS=$HOME/codes/nnstreamer/build/ext/nnstreamer/tensor_decoder
```

### Tensor Converter Plugins

```bash
export NNSTREAMER_CONVERTERS=$HOME/codes/nnstreamer/build/ext/nnstreamer/tensor_converter
```

---

## 9. Rebuild GStreamer Registry

Remove cached registry files:

```bash
rm -f ~/.cache/gstreamer-1.0/registry.*
```

This forces GStreamer to rescan available plugins.

---

## 10. Verify Tensor Filter Frameworks

Run:

```bash
gst-inspect-1.0 tensor_filter
```

Expected frameworks:

```text
Available frameworks:
   custom
   custom-easy
   tensorflow2-lite
   tvm
   cpp
```

---

## 11. Verify Tensor Decoder Modes

Run:

```bash
gst-inspect-1.0 tensor_decoder
```

Expected decoder modes:

```text
bounding_boxes
image_labeling
image_segment
direct_video
```

---

## 12. Run Installation Verification

Assuming the build directory is:

```text
./build
```

Run:

```bash
build/tools/development/confchk/nnstreamer-check
```

This performs a basic verification of the NNStreamer setup.

---

# Optional: Create a Reusable Environment Script

Instead of exporting variables manually every session, create a helper script.

## 1. Create Script

```bash
vi ~/env_nnstreamer.sh
```

## 2. Add the Following Content

```bash
export PKG_CONFIG_PATH=$HOME/codes/gstreamer/build/meson-uninstalled

export GST_PLUGIN_PATH=$HOME/codes/nnstreamer/build/gst/nnstreamer

export NNSTREAMER_CONF=$HOME/codes/nnstreamer/build/nnstreamer.ini

export NNSTREAMER_FILTERS=$HOME/codes/nnstreamer/build/ext/nnstreamer/tensor_filter

export NNSTREAMER_DECODERS=$HOME/codes/nnstreamer/build/ext/nnstreamer/tensor_decoder

export NNSTREAMER_CONVERTERS=$HOME/codes/nnstreamer/build/ext/nnstreamer/tensor_converter
```

## 3. Load the Environment

Whenever you start a new terminal session:

```bash
source ~/env_nnstreamer.sh
```

---

# Quick Verification Checklist

## GStreamer

```bash
gst-inspect-1.0 --version
```

## Tensor Filter

```bash
gst-inspect-1.0 tensor_filter
```

Verify:

```text
tensorflow2-lite
tvm
cpp
```

## Tensor Decoder

```bash
gst-inspect-1.0 tensor_decoder
```

Verify:

```text
bounding_boxes
image_labeling
image_segment
direct_video
```

## NNStreamer Validation

```bash
build/tools/development/confchk/nnstreamer-check
```

A successful execution confirms that NNStreamer is correctly configured.
