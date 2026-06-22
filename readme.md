# cmdfu - Microchip Device Firmware Update (MDFU) Host Tool

`cmdfu` is a lightweight, cross‑platform command‑line host application for performing firmware updates using the Microchip Device Firmware Update (MDFU) protocol. Written in C, cmdfu provides a native, dependency‑free alternative to Python‑based MDFU host tools, making it well suited for production environments, manufacturing, and embedded host systems. The tool implements the MDFU host side of the protocol, enabling reliable in‑field firmware updates for Microchip microcontrollers running an MDFU‑compatible bootloader. It communicates with target devices over supported transport layers (such as UART, SPI, or I²C, depending on configuration) and transfers application image files.

Key Features
- Native C implementation of an MDFU host
- Command‑line interface suitable for scripting and automation
- Implements the MDFU protocol command/response flow and error handling
- Designed for PC hosts or embedded hosts without Python runtime requirements

Supported interfaces
- Serial port (Windows/Linux)
- Linux SPI subsystem
- Linux I²C subsystem

cmdfu is provided as a reference host implementation to support the broader Microchip MDFU firmware update ecosystem.

## Configuration

Creating the build tree.
```bash
cmake -B build
```

### Configuration options

- MDFU_MAX_COMMAND_DATA_LENGTH: Defines the maximum MDFU command data length that is supported. This must be at least the same size as the MDFU client reported size.
- MDFU_MAX_RESPONSE_DATA_LENGTH: Defines the maximumd MDFU response data length that is supported.
- MDFU_LOG_TRANSPORT_FRAME: When defined and verbose level for logging is set to debug, the frames on the transport layer will be logged. Do not use for non-buffered mac layers or loggers since the logging can take some time depending on how this is implemented so the host might be late to receive the next frame after logging the sent frame.
- LINUX_SUBSYSTEM_I2C: Include Linux I2C target device, default ON.
- LINUX_SUBSYSTEM_SPI: Include Linux SPI target device, default ON.
- LINUX_SUBSYSTEM_SERIAL: Include Linux serial target device, default ON.
- WINDOWS_SUBSYSTEM_SERIAL: Include Windows serial target device, default OFF.
- LINUX_SUBSYSTEM_NETWORK: Include Linux network target device, default ON.

Example for creating the build tree and configuring maximum MDFU command data size.
```bash
cmake -B build -D MDFU_MAX_COMMAND_DATA_LENGTH=1024
```

Creating a build for Win32 serial devices only:
```bash
cmake -B build -D LINUX_SUBSYSTEM_I2C=OFF -D LINUX_SUBSYSTEM_SPI=OFF -D LINUX_SUBSYSTEM_NETWORK=OFF -D LINUX_SUBSYSTEM_SERIAL=OFF -D WINDOWS_SUBSYSTEM_SERIAL=ON
```

## Building

Building by invocing native build tools through cmake
```bash
cmake --build build
```
or by using native tools e.g. for make project
```bash
cd build
make
```

Build `cmdfu` utility only (needed for Windows build which doesn't yet support tests):
```bash
cmake --build build --target cmdfu
```

## Running the application from the build tree

```bash
cd .\build\apps\cmdfu\cmdfu
./cmdfu
```

## Installation from build tree

The install script will put the application into the system folders which are included in the search path.
```bash
cd ./build
make install
```

## Creating source and binary package

Create a package after building as a separate step.
```bash
cd build
cpack
```
The source and debian package will be available in the ./build directory.

Build and packaging in one step.
```bash
cmake --build build --target package
```
