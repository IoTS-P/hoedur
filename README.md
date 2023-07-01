# Hödur

Hoedur is a firmware fuzzing implementation which utilizes a multi-stream input format that is described in our USENIX Security 2023 paper `HOEDUR: Embedded Firmware Fuzzing using Multi-Stream Inputs`.

## Repository Overview

Hoedur consists of different main components as listed below:

| Directory | Description |
| --        | --          |
| emulator       | High-level emulator logic |
| fuzzer         | Hoedur fuzzer implementation |
| hoedur         | Command-line logic and runner |
| scripts        | Usability and evaluation scripts |
| modeling       | Integration with Fuzzware modeling |
| frametracer    | Trace events |
| hoedur-analyze | Utilities to evaluate fuzzing runs |
| archive        | Reading and writing fuzzing corpus archives |
| common         | Configurations and common utilities |
| qemu-build     | Qemu build, link, and interface code generation utility |
| qemu-rs        | Low-level emulator impl |
| qemu-sys       | Qemu rust bindings |

## Getting Started

### Dependencies
Ubuntu 18.04:
```sh
apt install -y clang curl git libfdt-dev libglib2.0-dev libpixman-1-dev libxcb-shape0-dev libxcb-xfixes0-dev ninja-build patchelf pkg-config python3-psutil zstd build-essential 
```

rust:
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

### Install

```sh
cargo install --path hoedur/ --bin hoedur-arm
sudo cp target/release/libqemu-system-arm.release.so /usr/lib/
```

### Development / Debug Build

Run a debug build (without install):
```sh
cargo run --bin hoedur-arm -- $ARGS
```

Run a release build (without install):
```sh
cargo run --bin hoedur-arm --release -- $ARGS
```

Chinese mirror:

Because **https://download.qemu.org** resolve failed in our server, so change the current lines in `qemu-sys/build.rs` 

change the `/path/to/qemu-..` to where you put your qemu archive 
```rust
// download QEMU
  //assert!(Command::new("wget")
   //   .arg("https://download.qemu.org/qemu-7.1.0.tar.xz")
    //  .arg("-O")
     // .arg(&qemu_tar)
     // .status()
      //.expect("QEMU download failed")
      //.success());

  assert!(Command::new("cp")
          .arg("/path/to/qemu-7.1.0.tar.xz")
          .arg(&qemu_tar)
          .status()
          .expect("no")
          .success());
```
## Fuzzer / Runner (Hoedur)

### Fuzzer

Basic usage:
```sh
CONFIG=arm/Hoedur/loramac-node/CVE-2022-39274/config.yml
cargo run --bin hoedur-arm -- --config $CONFIG fuzz
```

See help for details:
```sh
cargo run --bin hoedur-arm -- fuzz --help
```

### Runner

Run corpus archive:
```sh
ARCHIVE=corpus/hoedur.corpus.tar.zst
cargo run --bin hoedur-arm -- --import-config $ARCHIVE run-corpus $ARCHIVE
```

Run single input:
```sh
INPUT=corpus/input-123.bin
cargo run --bin hoedur-arm -- --import-config $ARCHIVE run $INPUT
```

### Coverage
Run fuzzer with `--statistics` enabled.

Collect coverage report from corpus archive:
```sh
REPORT=corpus/hoedur.report.bin.zst
hoedur-arm --debug --trace --import-config $ARCHIVE run-cov $REPORT $ARCHIVE
```

### Tracing / Hook Scripts

```sh
# run hoedur with a custom hook
# `--trace` enables tracing (will hook every basic block / instruction, needed for scripts)
hoedur-arm --import-config $ARCHIVE --debug --trace --hook example.rn run $INPUT
```
