---
layout: post
title: "Cross Compile OpenSSL for the Zynq ARM"
author: {{ site.author.name }}
---

## Prerequisites

### ARM GCC Toolchain

To follow this guide, you obviously need to have an ARM GCC toolchain on your machine. 

If you already have Xilinx's [PetaLinux](https://www.xilinx.com/products/design-tools/embedded-software/petalinux-sdk.html) installed, then you can find the toolchain at `<PETALINUX_INSTALL_DIR>/tools/linux-i386/gcc-arm-linux-gnueabi/bin`.

Otherwise, you can either:

1. Download the ARM GCC toolchain through your package manager; on Ubuntu, `sudo apt-get install gcc-arm-linux-gnueabihf`.
2. Download a build of the toolchain from [Linaro](https://releases.linaro.org/components/toolchain/binaries/latest/arm-linux-gnueabihf/). If you are on Linux, grab the i686 version (non-mingw32). You can find the actual binaries under the `bin/` directory after expanding the archive.

### Environment Setup

First, prepare some directories and clone the OpenSSL repo:

```bash
# Create working and installation directories
mkdir -p ~/cross-openssl/compiled-openssl && cd ~/cross-openssl

# Clone latest version of OpenSSL
git clone https://github.com/openssl/openssl.git
cd openssl/
```

Next, define environment variables to simplify the build process. If you installed the ARM GCC toolchain through your package manager, ignore the second variable.

```bash
# Installation directory for compiled OpenSSL
export INSTALL_DIR=~/cross-openssl/compiled-openssl

# Path to ARM cross compiler toolchain binaries
export CROSS_DIR=<PATH_TO_TOOLCHAIN>/bin
```

## Building OpenSSL

Now we are ready to configure the OpenSSL build system. For the following steps, please make sure that you are in the main OpenSSL directory; i.e., `pwd` should output `~/cross-openssl/openssl`.

```bash
./Configure linux-generic32 shared \
--prefix=$INSTALL_DIR --openssldir=$INSTALL_DIR/openssl \
--cross-compile-prefix=$CROSS_DIR/arm-linux-gnueabihf-
```

The command above:

1. Configures the OpenSSL build for a 32-bit Linux system (*linux-generic32*)
2. Generates both static and dynamic (shared) libraries
3. Installs the compiled OpenSSL folder in `INSTALL_DIR` (defined above)
4. And finally, sets the cross compiler prefix to `arm-linux-gnueabihf-`

**Note**: if you are using the cross compiler installed through your package manager, replace `--cross-compile-prefix=$CROSS_DIR/arm-linux-gnueabihf-` with `--cross-compile-prefix=arm-linux-gnueabihf-`.

Finally, we can build OpenSSL:

```bash
make depend
make # This takes time!
make install
```

The build process may take some time, depending on your machine. The resulting binary locations are as follows:

* openssl: `$INSTALL_DIR/bin/openssl`
* libcrypto: `$INSTALL_DIR/lib/libcrypto.[so/a]`
* libssl: `$INSTALL_DIR/lib/libssl.[so/a]`

Recall that, on Linux, the *.a* binaries are for **static** linking, while the *.so* files are for dynamic linking. The easier approach is to use the former, but your final binary size will be quite large as a result.
