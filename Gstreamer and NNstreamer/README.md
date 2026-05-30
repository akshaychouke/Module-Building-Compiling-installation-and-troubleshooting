Building the Gstreamer from sources

(Pre-requesites : Please install all the required dependencies)

1. Clone gstreamer from Gitlab
      git clone https://gitlab.freedesktop.org/gstreamer/gstreamer.git
2. cd gstreamer
3. Checkout to your desired branch
      git checkout 1.24
4. Build, Compileand install gstreamer
      meson setup build
      ninja -C build
      sudo ninja -C build install (This will install gstreamer in /usr/local/ by default. It is optional)
5. Start gstreamer environment
      ./gst-env.py

6. Check gstreamer installation
      gst-inspect-1.0 --version


/////////////////////////////////

Building the nnstreamer from sources using above built gstreamer

(Pre-requisite: gstreamer should be built and installed in the system before building nnstreamer or if not installed then the gstreamer environment must be active)

Install the required dependencies ->

sudo add-apt-repository ppa:nnstreamer/ppa
sudo apt-get update
sudo apt-get install libedgetpu-dev libflatbuffers-dev libgrpc-dev openvino-dev libpaho-mqtt-dev libprotobuf-dev pytorch libopencv-dev tensorflow2-lite-dev tvm-runtime-dev

sudo apt install \
    libglib2.0-dev \
    libjson-glib-dev \
    python3-dev \
    python3-numpy \
    flex \
    bison


1. Clone nnstreamer from Github
      git clone https://gitlab.com/nnstreamer/nnstreamer.git
2. cd nnstreamer
3. Checkout to your desired branch
      git checkout 1.14
4. Build, Compile and install nnstreamer
      meson setup build
      meson compile -C build

      sudo ninja -C build install (This will install nnstreamer in /usr/local/ by default. It is optional)

5. Add the nnstreaner in GST_PLUGIN_PATH
      export GST_PLUGIN_PATH="${GST_PLUGIN_PATH:+$GST_PLUGIN_PATH:}$HOME/codes/nnstreamer/build/gst/nnstreamer"

6. Verify subplugin libraries were generated
      find build/ext/nnstreamer/tensor_filter -name "*.so"
   
   You should see things like below, which means TensorFlow Lite support was successfully built ->

   libnnstreamer_filter_tensorflow2-lite.so
   libnnstreamer_filter_tvm.so

7. Tell NNStreamer where the config file is ->
   i. NNStreamer generates: build/nnstreamer.ini

   ii. Export:
      export NNSTREAMER_CONF=$HOME/codes/nnstreamer/build/nnstreamer.ini
   
   iii Without this: Error like -> Failed to load configuration, no config file found and framework subplugins won't be discovered.


8. Tell NNStreamer where subplugins are ->

   i. For source builds: 

      export NNSTREAMER_FILTERS=$HOME/codes/nnstreamer/build/ext/nnstreamer/tensor_filter

      export NNSTREAMER_DECODERS=$HOME/codes/nnstreamer/build/ext/nnstreamer/tensor_decoder

      export NNSTREAMER_CONVERTERS=$HOME/codes/nnstreamer/build/ext/nnstreamer/tensor_converter

9. Rebuild GStreamer registry ->
      rm -f ~/.cache/gstreamer-1.0/registry.*

10. Verify tensor_filter frameworks ->
   i. Run:  gst-inspect-1.0 tensor_filter

   ii. Look for below which confirms everything is loaded correctly:
         Available frameworks:
            custom
            custom-easy
            tensorflow2-lite
            tvm
            cpp

11. Verify tensor_decoder modes 
    i. Run : gst-inspect-1.0 tensor_decoder
    ii. Look for:
         bounding_boxes
         image_labeling
         image_segment
         direct_video

12. Verifying installation ->
   Assuming you have configured meson at ./build directory -> 
      build/tools/development/confchk/nnstreamer-check



Optional ->

1. Create a reusable environment script->

      vi ~/env_nnstreamer.sh

2. Copy past the below

      export PKG_CONFIG_PATH=$HOME/codes/gstreamer/build/meson-uninstalled

      export GST_PLUGIN_PATH=$HOME/codes/nnstreamer/build/gst/nnstreamer

      export NNSTREAMER_CONF=$HOME/codes/nnstreamer/build/nnstreamer.ini

      export NNSTREAMER_FILTERS=$HOME/codes/nnstreamer/build/ext/nnstreamer/tensor_filter

      export NNSTREAMER_DECODERS=$HOME/codes/nnstreamer/build/ext/nnstreamer/tensor_decoder

      export NNSTREAMER_CONVERTERS=$HOME/codes/nnstreamer/build/ext/nnstreamer/tensor_converter


3. Load it whenever needed:
      source ~/env_nnstreamer.sh
