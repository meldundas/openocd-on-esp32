# OpenOCD on ESP32-S3

This project is an ESP-IDF application that ports OpenOCD to run directly on an ESP32-S3. It transforms the ESP32-S3 into a stand-alone, network-attached hardware debugger capable of debugging other microcontrollers via JTAG or SWD.

## Project Overview

- **Purpose**: Stand-alone debugger with GDB server access over Wi-Fi/Ethernet.
- **Core Technology**: ESP-IDF (v5.0+), OpenOCD (v0.12.0), JimTCL.
- **Supported Hardware**: ESP32-S3 with PSRAM (mandatory due to memory requirements).
- **Target Support**: Pre-configured for Espressif chips (ESP32, S2, S3, C-series) and extendable to others (e.g., STM32 via custom config files).
- **Architecture**:
    - **OpenOCD Component**: The full OpenOCD source is integrated and compiled for Xtensa.
    - **FATFS Storage**: OpenOCD TCL scripts are stored in a FATFS partition (mounted at `/data`).
    - **Web Interface**: A built-in web server allows runtime configuration of Wi-Fi, target scripts, and debug interfaces.
    - **UI (Optional)**: Support for ESP-BOX touchscreen configuration.

## Key Files

- `main/main.c`: Application entry point; initializes networking, storage, and launches the OpenOCD task.
- `main/openocd/`: Submodule/Directory containing the OpenOCD source code.
- `main/openocd/tcl-lite/`: Pruned set of OpenOCD scripts flashed to the device's FATFS partition.
- `sdkconfig.defaults`: Default configuration including Flash size, PSRAM mode, and OpenOCD default target/interface.
- `partitions_example.csv`: Custom partition table defining the storage for TCL scripts.

## Building and Running

### Prerequisites
- ESP-IDF v5.0 or newer installed and sourced.
- ESP32-S3 board with PSRAM (e.g., ESP32-S3-WROOM-1-N16R8).

### Commands
- **Build**: `idf.py build`
- **Flash**: `idf.py flash` (Note: This flashes the application and generates/flashes the FATFS image).
- **Monitor**: `idf.py monitor`
- **Clean**: `idf.py clean`
- **Menuconfig**: `idf.py menuconfig` (Use this to configure Wi-Fi SSID/Password and OpenOCD target defaults).

## Development Conventions

- **OpenOCD Interface**: Default pins are GPIO 38 (TCK/SWCLK), 39 (TMS/SWDIO), 40 (TDI), 41 (TDO). These can be changed in `menuconfig` or at runtime via the Web UI.

The ESP32-S3 has been successfully flashed with the new configuration for the
  Nucleo F446 (STM32F4x) using the SWD interface.

Updated SWD Connections (including Reset):
   * ESP32 GPIO 38 -> Nucleo SWCLK (PA14)
   * ESP32 GPIO 39 -> Nucleo SWDIO (PA13)
   * ESP32 GPIO 40 -> Nucleo NRST (System Reset)
   * GND -> Nucleo GND

  The firmware is now running OpenOCD with target/stm32f4x.cfg. You can now
  connect your Nucleo board and start debugging!

- **Target Configuration**: Target scripts must be placed in `main/openocd/tcl-lite/target/` to be available on the device.
- **Adding New Targets**: 
    1. Copy the `.cfg` file to `main/openocd/tcl-lite/target/`.
    2. Ensure dependencies (e.g., `swj-dp.tcl`) are also in `tcl-lite`.
    3. Update `sdkconfig.defaults` or use the Web UI to select the new config.

## Future Interactions Context

When helping with this project, prioritize:
1.  **Hardware Compatibility**: Ensure Flash (e.g., 16MB) and PSRAM (e.g., Octal) settings match the specific ESP32-S3 module.
2.  **Network Setup**: The device defaults to AP mode (`esp-openocd`) if credentials are not set.
3.  **TCL Script Management**: Remember that only scripts in `tcl-lite` are accessible to the running OpenOCD instance.
