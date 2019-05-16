Introduction
==============
LittlevGL is an Open-source Embedded GUI Library. We defined a UI APP, which can easily run on Native or on WASM VM. There are 3 binaries to test two scenarios.
1. Native Linux. The App code built into Linux executables.
2. WASM VM for Different platforms. WASM VM and native extension being built into Linux and Zephyr platforms. With WASM VM inside, many WASM APP can run on top of it.
3. WASM APP. This kind of binary can be run on WASM VM.

Directory structure
--------------------------------
<pre>
├── build.sh
├── LICENCE.txt
├── README.md
├── UI.JPG
├── vgl-native-ui-app
│   ├── CMakeLists.txt
│   ├── lv-drivers
│   │   ├── display_indev.h
│   │   ├── indev
│   │   │   ├── mouse.c
│   │   │   └── mouse.h
│   │   ├── linux_display_indev.c
│   │   ├── lv_conf.h
│   │   └── system_header.h
│   └── main.c
├── vgl-wasm-runtime
│   ├── CMakeLists.txt
│   ├── src
│   │   ├── display_indev.h
│   │   ├── ext-lib-export.c
│   │   └── platform
│   │       ├── linux
│   │       │   ├── display_indev.c
│   │       │   ├── iwasm_main.c
│   │       │   ├── main.c
│   │       │   └── mouse.c
│   │       └── zephyr
│   │           ├── board_config.h
│   │           ├── display.h
│   │           ├── display_ili9340_adafruit_1480.c
│   │           ├── display_ili9340.c
│   │           ├── display_ili9340.h
│   │           ├── display_indev.c
│   │           ├── iwasm_main.c
│   │           ├── LICENSE
│   │           ├── main.c
│   │           ├── pin_config_jlf.h
│   │           ├── pin_config_stm32.h
│   │           ├── XPT2046.c
│   │           └── XPT2046.h
│   └── zephyr-build
│       ├── CMakeLists.txt
│       └── prj.conf
└── wasm-apps
    ├── build_wasm_app.sh
    ├── Makefile_wasm_app
    ├── src
    │   ├── display_indev.h
    │   ├── lv_conf.h
    │   ├── main.c
    │   └── system_header.h
    └── ui_app.wasm
</pre>
- build.sh
  This build to build and binaries.
- LICENCE.txt
- UI.JPG
- user_guide.md
- vgl-native-ui-app
  LittlevGL graphics app has been built into Linux application named "vgl_native_ui_app", which can directly run on Linux.
- vgl-wasm-runtime
  Wasm micro-runtime and Littlevgl native interface built into Linux application named "LittlevGL", where the WASM application can run on it.
- wasm-apps
  A wasm app with Littlevgl graphics.


Install required SDK and libraries
==============
- 32 bit SDL(simple directmedia layer)
<pre>
Use apt-get
    sudo apt-get install libsdl2-dev:i386
Or, install from source  
   Download source from www.libsdl.org
    `./configure C_FLAGS=-m32 CXX_FLAGS=-m32 LD_FLAGS=-m32`
     ` make`
    `sudo make install`
</pre>
- Install EMSDK
<pre>
    https://emscripten.org/docs/tools_reference/emsdk.html
</pre>
- Cmake
<pre>
     CMAKE version above 3.13.1.
</pre>

Build & Run
==============

Build and run on Linux
--------------------------------
- Build
`./build.sh`
    All binaries are in "out", which contains "host_tool", "vgl_native_ui_app", "TestApplet1.wasm" and "vgl_wasm_runtime".
- Run native Linux application
`./vgl_native_ui_app`
<pre>
<img src="./UI.JPG">
The number on top will plus one each second, and the number on the bottom will plus one when clicked.
</pre>
- Run WASM VM Linux applicaton & install WASM APP
 First start vgl_wasm_runtime in server mode.
`./vgl_wasm_runtime -s`
 Then install wasm APP use host tool.
`./host_tool -i TestApplet1 -f TestApplet1.wasm`


Build and run on Zephyr
--------------------------------
WASM VM and native extension method can be built into Zephyr, Then we can install wasm app into STM32.
- Build WASM VM into Zephyr system
 a. clone zephyr source code
`git clone https://github.com/zephyrproject-rtos/zephyr.git`
 b. copy samples
    `cd zephyr/samples/`
    `cp -a <iwasm_root_dir>samples/littlevgl/vgl-wasm-runtime vgl-wasm-runtime`
    `cd vgl-wasm-runtime/zephyr_build`
 c. create a link to wamr core
   ` ln -s <iwasm_root_dir>/core core`
 d. build source code
    Since ui_app incorporated LittlevGL source code, so it needs more RAM on the device to install the application.
    It is recommended that RAM SIZE greater than 512KB.
    In our test use nucleo_f767zi, which is not supported by Zephyr.
    However, nucleo_f767zi is almost the same as nucleo_f746zg, except FLASH and SRAM size.
    So we changed the DTS setting of nucleo_f746zg boards for a workaround.

    `Modify zephyr/dts/arm/st/f7/stm32f746xg.dtsi, change DT_SIZE_K(320) to DT_SIZE_K(512)`
    `mkdir build && cd build`
    `source ../../../../zephyr-env.sh`
    `cmake -GNinja -DBOARD=nucleo_f746zg ..`
   ` ninja flash`

- Test on STM32 NUCLEO_F767ZI with ILI9341 Display with XPT2046 touch.
Hardware Connections
<pre>
+-------------------+-+------------------+
|NUCLEO-F767ZI || ILI9341  Display |
+-------------------+-+------------------+
| CN7.10               |         CLK     |
+-------------------+-+------------------+
| CN7.12               |         MISO    |
+-------------------+-+------------------+
| CN7.14               |         MOSI    |
+-------------------+-+------------------+
| CN11.1               | CS1 for ILI9341 |
+-------------------+-+------------------+
| CN11.2               |         D/C     |
+-------------------+-+------------------+
| CN11.3               |         RESET   |
+-------------------+-+------------------+
| CN9.25               |    PEN interrupt|
+-------------------+-+------------------+
| CN9.27               | CS2 for XPT2046 |
+-------------------+-+------------------+
| CN10.14             | PC UART RX       |
+-------------------+-+------------------+
| CN11.16             | PC UART RX       |
+-------------------+-+------------------+
</pre>
- Install WASM application to Zephyr using host_tool
First, connect PC and STM32 with UART. Then install to use host_tool.
`./host_tool -D /dev/ttyUSBXXX -i ui_app -f ui_app.wasm`

