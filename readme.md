# cmdfu - Microchip Device Firmware Update (MDFU) Host Tool

`cmdfu` is a lightweight, cross-platform command-line host application for performing firmware updates using the Microchip Device Firmware Update (MDFU) protocol. Written in C, `cmdfu` provides a native, dependency-free alternative to Python-based MDFU host tools, making it well suited for production environments, manufacturing, and embedded host systems.

The tool implements the MDFU host side of the protocol, enabling reliable in-field firmware updates for Microchip microcontrollers running an MDFU-compatible bootloader. It communicates with target devices over UART, SPI, or I2C (depending on build configuration) and transfers application image files.

## Key Features

- Native C implementation with no runtime dependencies
- Command-line interface designed for scripting and automation
- Full MDFU protocol command/response flow with error handling and retries
- Layered architecture: protocol, transport, and MAC layers are cleanly separated
- Suitable for PC hosts and embedded Linux hosts without a Python runtime

## Supported Interfaces

| Interface | Platform |
|---|---|
| Serial (UART) | Windows, Linux |
| SPI (Linux spidev) | Linux |
| I2C (Linux i2cdev) | Linux |
| Network (TCP socket) | Linux (for simulation/testing) |

## Usage

```
cmdfu [-h | --help] [-v <level> | --verbose <level>] [-V | --version] [-R | --release-info] <action>
```

### Actions

| Action | Description |
|---|---|
| `update` | Perform a firmware update |
| `client-info` | Query MDFU client information from the target device |
| `tools-help` | Show parameter help for all compiled-in tools |

### Global Options

| Option | Description |
|---|---|
| `-h`, `--help` | Show help (context-sensitive per action) and exit |
| `-V`, `--version` | Print cmdfu version number and exit |
| `-R`, `--release-info` | Print cmdfu version and MDFU protocol version and exit |
| `-v <level>`, `--verbose <level>` | Set logging verbosity: `error`, `warning`, `info`, `debug`. Default: `error` |
| `-t <tool>`, `--tool <tool>` | Select the communication tool |

### `update` Action

Perform a firmware update on a connected MDFU target device.

```
cmdfu [--verbose <level>] update --tool <tool> --image <image> [<tool-args>...]
```

| Option | Description |
|---|---|
| `--tool <tool>`, `-t <tool>` | Communication tool to use (required) |
| `--image <file>`, `-i <file>` | Path to the firmware image file (required) |

### `client-info` Action

Query and print MDFU client information (protocol version, buffer sizes, timeouts, inter-transaction delay).

```
cmdfu [--verbose <level>] client-info --tool <tool> [<tool-args>...]
```

### Tool Parameters

Use `cmdfu tools-help` to list all tool-specific parameters. Common parameters per tool:

**`serial`** (Windows and Linux)
```
--port <port>        Serial port, e.g. /dev/ttyACM0 or COM11
--baudrate <baud>    Baud rate (default: 115200)
```

Supported baud rates: 9600, 19200, 38400, 57600, 115200, 230400, 460800, 500000, 576000, 921600, 1000000, 1152000, 1500000, 2000000, 2500000, 3000000, 3500000, 4000000.

**`spidev`** (Linux only)
```
--dev <device>       SPI device, e.g. /dev/spidev0.0
--clk-speed <hz>     Clock speed in Hz, e.g. 1000000
--mode <mode>        SPI mode: 0, 1, 2, or 3
```

**`i2cdev`** (Linux only)
```
--dev <device>       I2C device, e.g. /dev/i2c-0
--address <addr>     I2C target address (0-127)
```

**`network`** (Linux only — for simulation/testing over TCP)
```
--host <host>        Remote host (default: localhost)
--port <port>        TCP port (default: 5559)
--transport <type>   Transport type: serial, serial-buffered, spi, i2c
```

### Examples

```bash
# Firmware update over serial on Linux
cmdfu update --tool serial --image update_image.img --port /dev/ttyACM0 --baudrate 115200

# Firmware update over serial on Windows
cmdfu update --tool serial --image update_image.img --port COM11 --baudrate 115200

# Query client info over I2C
cmdfu client-info --tool i2cdev --dev /dev/i2c-0 --address 55

# Firmware update over SPI
cmdfu update --tool spidev --image fw.img --dev /dev/spidev0.0 --clk-speed 1000000 --mode 0

# Enable debug logging
cmdfu -v debug update --tool serial --image fw.img --port /dev/ttyACM0
```

## Configuration

Generate the build tree:
```bash
cmake -B build
```

### Configuration Options

| Option | Default | Description |
|---|---|---|
| `MDFU_MAX_COMMAND_DATA_LENGTH` | `1024` | Maximum MDFU command data buffer size. Must be at least the size reported by the MDFU client. |
| `MDFU_MAX_RESPONSE_DATA_LENGTH` | `30` | Maximum MDFU response data buffer size. |
| `MDFU_LOG_TRANSPORT_FRAME` | undefined | When defined and verbosity is `debug`, raw transport frames are logged. Do not use with non-buffered MAC layers as logging latency may cause missed frames. |
| `LINUX_SUBSYSTEM_I2C` | `ON` | Include the Linux I2C tool and MAC driver. |
| `LINUX_SUBSYSTEM_SPI` | `ON` | Include the Linux SPI tool and MAC driver. |
| `LINUX_SUBSYSTEM_SERIAL` | `ON` | Include the Linux serial tool and MAC driver. |
| `LINUX_SUBSYSTEM_NETWORK` | `ON` | Include the Linux network tool and socket MAC driver. |
| `WINDOWS_SUBSYSTEM_SERIAL` | `OFF` | Include the Windows serial tool (Win32 API). |

Example: configure with a custom maximum command data size:
```bash
cmake -B build -D MDFU_MAX_COMMAND_DATA_LENGTH=1024
```

Example: configure for Windows serial only:
```bash
cmake -B build -D LINUX_SUBSYSTEM_I2C=OFF -D LINUX_SUBSYSTEM_SPI=OFF -D LINUX_SUBSYSTEM_NETWORK=OFF -D LINUX_SUBSYSTEM_SERIAL=OFF -D WINDOWS_SUBSYSTEM_SERIAL=ON
```

## Building

Build using CMake:
```bash
cmake --build build
```

Or using the native build tool directly (e.g. make):
```bash
cd build
make
```

Build only the `cmdfu` executable (required for Windows, which does not support the test targets):
```bash
cmake --build build --target cmdfu
```

## Running from the Build Tree

**Linux:**
```bash
./build/apps/cmdfu/cmdfu
```

**Windows:**
```
.\build\apps\cmdfu\cmdfu.exe
```

## Installation

Install the `cmdfu` binary to the system path:
```bash
cd build
make install
```

Or via CMake:
```bash
cmake --install build
```

## Packaging

Create a binary and source package after building:
```bash
cd build
cpack
```

Build and package in one step:
```bash
cmake --build build --target package
```

Packages produced: TGZ binary, DEB (Debian), and TBZ2 source archive.

## Testing

Unit tests use the [Ceedling](https://github.com/ThrowTheSwitch/Ceedling) framework. Tests are supported on Linux only and are run inside a Docker container:

```bash
docker run --rm -v $(pwd):/workspace throwtheswitch/madsciencelab:1.0.8 /bin/bash -c "cd /workspace && ceedling test:all"
```

> Note: Windows builds do not include test targets. Use `--target cmdfu` when building on Windows.
