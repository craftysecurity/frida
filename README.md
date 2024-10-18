# Changes 
The memmem changes fix the issues where frida looks for AUXV from the bottom of the stack which causes asan issues when run with afl-frida-trace by implementing a static memmem and downgrading frida-gum to 16.3.0:

Address 0x008000000000 is a wild pointer inside of access range of size 0x000000021000.
SUMMARY: AddressSanitizer: unknown-crash (/data/local/tmp/fasan/libclang_rt.asan-aarch64-android.so+0x7e39c) (BuildId: 1a99d2d2b43648db69abf23476a24d458d41ad00)
==10021==ABORTING

# Compiling afl-fuzz/afl-frida-trace and frida-gum devkit

First compile the frida-gumjs devkit from the frida root directory

git clone https://github.com/craftysecurity/frida.git
make clean
./configure --host=android-arm64 -- \
    -Dfrida-gum:devkits=gum,gumjs \
    -Dfrida-core:devkits=core
make

Clone aflplusplus 4.0.6
git clone https://github.com/craftysecurity/AFLplusplus/tree/stable

CMakeLists.txt to compile afl-fuzz and afl-frida-trace 
https://gist.github.com/craftysecurity/29f6f78d5368246542bda572dd405242

mkdir build && cd build
cmake -DANDROID_PLATFORM=34 \
        -DCMAKE_TOOLCHAIN_FILE=/opt/android-ndk-r25c/build/cmake/android.toolchain.cmake \
        -DANDROID_ABI=arm64-v8a ..
make

# Frida

Dynamic instrumentation toolkit for developers, reverse-engineers, and security
researchers. Learn more at [frida.re](https://frida.re/).

Two ways to install
===================

## 1. Install from prebuilt binaries

This is the recommended way to get started. All you need to do is:

    pip install frida-tools # CLI tools
    pip install frida       # Python bindings
    npm install frida       # Node.js bindings

You may also download pre-built binaries for various operating systems from
Frida's [releases](https://github.com/frida/frida/releases) page on GitHub.

## 2. Build your own binaries

Run:

    make

You may also invoke `./configure` first if you want to specify a `--prefix`, or
any other options.

### CLI tools

For running the Frida CLI tools, e.g. `frida`, `frida-ls-devices`, `frida-ps`,
`frida-kill`, `frida-trace`, `frida-discover`, etc., you need a few packages:

    pip install colorama prompt-toolkit pygments

### Apple OSes

First make a trusted code-signing certificate. You can use the guide at
https://sourceware.org/gdb/wiki/PermissionsDarwin in the sections
“Create a certificate in the System Keychain” and “Trust the certificate
for code signing”. You can use the name `frida-cert` instead of `gdb-cert`
if you'd like.

Next export the name of the created certificate to relevant environment
variables, and run `make`:

    export MACOS_CERTID=frida-cert
    export IOS_CERTID=frida-cert
    export WATCHOS_CERTID=frida-cert
    export TVOS_CERTID=frida-cert
    make

To ensure that macOS accepts the newly created certificate, restart the
`taskgated` daemon:

    sudo killall taskgated

## Learn more

Have a look at our [documentation](https://frida.re/docs/home/).
