# Usage

- Clone the [platform repo](https://github.com/dashpay/platform)
- Place kotlin-dpp folder inside [platform/packages](https://github.com/dashpay/platform/tree/v0.24.5/packages)
- Setup Rust + NDK using [this guide](https://sal.dev/android/intro-rust-android-uniffi/) (step 1).
- Generate Rust libraries using this command (you can omit --release, but the debug libraries will be very big).

```cargo build --target x86_64-linux-android --target i686-linux-android --target armv7-linux-androideabi --target aarch64-linux-android --release```

- Copy generated libraries into kotlin-dpp. This repo already has the generated libraries, remove them if nessesary.
```
mkdir -p jniLibs/arm64-v8a/ && \
  cp ../../target/aarch64-linux-android/release/libdpp.so jniLibs/arm64-v8a/libuniffi_kotlin_dpp.so && \
  mkdir -p jniLibs/armeabi-v7a/ && \
    cp ../../target/armv7-linux-androideabi/release/libdpp.so jniLibs/armeabi-v7a/libuniffi_kotlin_dpp.so && \
  mkdir -p jniLibs/x86/ && \
    cp ../../target/i686-linux-android/release/libdpp.so jniLibs/x86/libuniffi_kotlin_dpp.so && \
  mkdir -p jniLibs/x86_64/ && \
    cp ../../target/x86_64-linux-android/release/libdpp.so jniLibs/x86_64/libuniffi_kotlin_dpp.so
```

- Uncomment the following code in Cargo.toml (to avoid this step, we'll need to move `uniffi-bindgen` to [its own crate](https://mozilla.github.io/uniffi-rs/tutorial/foreign_language_bindings.html#multi-crate-workspaces) later):

```
#[[bin]]
#name = "uniffi-bindgen"
#path = "uniffi-bindgen.rs"
```

- Run this command to generate Kotlin bindings for the libraries:

```cargo run --features=uniffi/cli --bin uniffi-bindgen generate src/dpp.udl --language kotlin```

- Copy `jniLibs` into wallet/src/main of the wallet repo.
- Copy `src/uniffi/kotlin_dpp/kotlin_dpp.kt` into the wallet repo.

Example of using the bindings:

`Log.i("DPP", "systemIds: ${systemIds()}")`

# Possible issues

```is aarch64-linux-android-ar installed?```

Can be solved by making copies of `llvm-ar` in the NDK folder as `aarch64-linux-android-ar` and other architectures: https://stackoverflow.com/questions/69945638/do-rust-and-cargo-1-56-simply-not-work-with-android-ndk-r23-without-binutils

```i686-linux-android-clang: error: not ranlib, ar, lib or dlltool```

Similary to above, copy `i686-linux-android23-clang` as `i686-linux-android-clang`

`i686-linux-android21-clang failed to execute` - remove lower API binaries from the toolchain folder.

There might be more elegant solutions to these issues.
