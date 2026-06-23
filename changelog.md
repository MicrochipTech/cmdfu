# cmdfu Changelog

All notable changes to this project will be documented in this file.

## 0.4.0

### Added

- MDFU-21: GitHub Actions workflow for multi-OS (Ubuntu, Windows) and multi-compiler (GCC, Clang, MSVC) builds
- MDFU-9: README with build, configuration, and usage documentation
- MDFU-26: Add this changelog

### Fixed

- MDFU-25: Reset inter-transaction delay on I2C transport when delay values are updated
- MDFU-34: Command line help fixes
- MDFU-35: Fixed test builds

## 0.3.0

### Added

- Windows Win32 serial port support (`WINDOWS_SUBSYSTEM_SERIAL` CMake option)

### Fixed

- MDFU-1: Pin the Ceedling Docker container to a working version to fix unit test execution
- MDFU-2: Do not report an error on I2C transport write
- MDFU-3: Align include paths for MDFU configuration header with the rest of the project
- MDFU-7: Buffer optimization to reduce memory usage
- MDFU-10: Validate the frame size of received frames on the serial transport
- MDFU-13: On command retry, do not reuse the command packet size for the response buffer size
- MDFU-14: Serial transport frame logging no longer blocks polled serial data reception
- MDFU-16: `WriteChunk` command no longer sends an empty data payload

## 0.2.0

Initial release.
