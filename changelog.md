# cmdfu changelog

## 0.4.0

### Added

- MDFU-21: Github workflow for multi OS and compiler builds
- MDFU-9: Readme with build and configuration instructions
- MDFU-26: Add a changelog 

### Fixed

- MDFU-25: Reset inter transaction delay on I2C transport on delay updates

## 0.3.0

### Fixed
- MDFU-1: Fix unit testing by using a working version of the ceedling docker container
- MDFU-2: Do not report error on I2C transport write
- MDFU-3: Align include paths for mdfu configuration with other includes 
- MDFU-7: Buffer optimization
- MDFU-10: Validate frame size for a received frame on serial transport
- MDFU-13: On command retry do not re-use command package size for response size
- MDFU-14: Serial transport frame logging blocks polled serial data receival
- MDFU-16: Write chunk command should not send empty data


## 0.2.0 Initial release