# Development Setup

## Prerequisites

- **CMake** 3.12 or later (CI uses 3.25+)
- **C compiler**: GCC or Clang on Linux; GCC (MinGW) or MSVC on Windows
- **Docker** (recommended for running unit tests)
- **VSCode** with the C/C++ extension (optional but recommended)

---

## Building

### Configure

```bash
# Default Linux build — all subsystems enabled
cmake -S . -B build

# Generate compile_commands.json for IDE IntelliSense (recommended)
cmake -S . -B build -DCMAKE_EXPORT_COMPILE_COMMANDS=ON

# Windows serial-only build
cmake -S . -B build \
  -D LINUX_SUBSYSTEM_I2C=OFF \
  -D LINUX_SUBSYSTEM_SPI=OFF \
  -D LINUX_SUBSYSTEM_NETWORK=OFF \
  -D LINUX_SUBSYSTEM_SERIAL=OFF \
  -D WINDOWS_SUBSYSTEM_SERIAL=ON
```

### Build

```bash
# Build everything (Linux)
cmake --build build

# Build only the cmdfu executable (required on Windows)
cmake --build build --target cmdfu
```

### CMake link order

When linking static libraries the order matters — dependents must be listed before their dependencies:

```cmake
target_link_libraries(cmdfu PRIVATE mdfulib toolslib transportlib maclib utilslib)
```

## Linux Kernel Headers

Some subsystems require kernel userspace headers that may not be present in all environments.

### Raspberry Pi

```bash
apt install raspberrypi-kernel-headers
```

### WSL2

WSL2 does not provide installable kernel header packages through the distribution's package manager. Headers must be generated from the WSL2 kernel source.

1. Clone the WSL2 kernel repository, matching the branch to your running kernel version:

```bash
git clone --depth 1 --branch linux-msft-wsl-5.15.y \
  https://github.com/microsoft/WSL2-Linux-Kernel.git
```

Verify the branch matches your kernel:
```bash
uname -r
# e.g. 5.15.146.1-microsoft-standard-WSL2
```

2. Install the tools required to configure the kernel:

```bash
apt-get install bison flex libncurses-dev
```

3. Configure the kernel to export headers:

```bash
make KCONFIG_CONFIG=Microsoft/config-wsl menuconfig
```

In the menu, enable `CONFIG_HEADERS_INSTALL`:
```
Linux Kernel Configuration
└─> Kernel hacking
    └─> Compile-time checks and compiler options
        └─> Install uapi headers to usr/include
```

4. Generate and install the headers:

```bash
make headers_install INSTALL_HDR_PATH=/usr/include
```

If you install to a non-system path, add it to `"includePath"` in `.vscode/c_cpp_properties.json`:

```json
{
    "configurations": [
        {
            "name": "Linux-WSL",
            "includePath": [
                "${workspaceFolder}/include/**",
                "${workspaceFolder}/build/include/**",
                "/path/to/wsl/kernel/headers/**"
            ],
            "compilerPath": "/usr/bin/gcc",
            "cStandard": "gnu23",
            "intelliSenseMode": "linux-gcc-x64",
            "compileCommands": "${workspaceFolder}/build/compile_commands.json"
        }
    ],
    "version": 4
}
```

After updating `c_cpp_properties.json`, reload the VSCode window (`Ctrl+Shift+P` → "Reload Window").

---

## Unit Tests

Tests use the [Ceedling](https://github.com/ThrowTheSwitch/Ceedling) framework (Unity + CMock). Tests are **Linux-only**. The recommended method is Docker.

The CMake build tree must be configured first — Ceedling's source paths include `../build/include` for the generated header `mdfu_config.h`:

```bash
cmake -S . -B build
```

### Docker (recommended)

```bash
docker run --rm \
  -v $(pwd):/home/dev/project \
  --user $(id -u):$(id -g) \
  throwtheswitch/madsciencelab:1.0.1b \
  /bin/bash -c "cd /home/dev/project/test && rm -rf build && ceedling test:all"
```

To run a single test:
```bash
docker run --rm \
  -v $(pwd):/home/dev/project \
  --user $(id -u):$(id -g) \
  throwtheswitch/madsciencelab:1.0.1b \
  /bin/bash -c "cd /home/dev/project/test && rm -rf build && ceedling test:mdfu_client_info"
```

**Notes:**
- `--user $(id -u):$(id -g)` matches the container process UID to the host filesystem owner, avoiding permission errors when Ceedling creates `test/build/`.
- `rm -rf build` removes any stale Ceedling build output from a previous run.
- The image tag `1.0.1b` is the latest published `madsciencelab` tag. Do not change it without verifying tests still pass.

### Local Ceedling

Requires Ruby (any version supported by Ceedling 1.0.x) and the Ceedling gem:

```bash
gem install ceedling
```

From the `test/` directory:
```bash
ceedling test:all
ceedling test:mdfu_client_info
ceedling test:serial_transport_layer
```

---

## CI/CD

### Jenkins

The Jenkins pipeline is defined in `cicd/Jenkinsfile`. It runs on a Kubernetes pod defined in `cicd/cloudprovider.yaml`, which provisions two containers:

| Container | Image | Purpose |
|---|---|---|
| `buildpack-deps` | `buildpack-deps:stable` | CMake configure and build |
| `ceedling` | `throwtheswitch/madsciencelab:1.0.1b` | Ceedling unit tests |

Pipeline stages:

1. **Install Dependencies** — installs CMake into `buildpack-deps`
2. **Build** — configures CMake and builds the `cmdfu` executable
3. **Test** — runs `ceedling test:all` in the `ceedling` container; test failures do not abort the pipeline so that the JUnit report is always published

The JUnit XML report is written to `test/build/artifacts/test/test_report.xml` and published via the Jenkins JUnit plugin.

### GitHub Actions

Two workflows are defined in `.github/workflows/`:

**`build.yml`** — triggered on push and pull requests to `main`, and manually via `workflow_dispatch`:

| OS | Compiler | Notes |
|---|---|---|
| ubuntu-latest | gcc | All Linux subsystems enabled |
| ubuntu-latest | clang | All Linux subsystems enabled |
| windows-latest | gcc (MinGW) | Windows serial only, `--target cmdfu` |

Build artifacts are uploaded as `cmdfu-<OS>-<compiler>.zip`.

**`cmake-multi-platform.yml`** — additional matrix including MSVC on Windows. No artifact upload.
