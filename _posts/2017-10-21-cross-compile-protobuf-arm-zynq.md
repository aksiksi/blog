---
layout: post
title: "Cross Compile Protobuf for the Zynq ARM"
author: {{ site.author.name }}
---

## Prerequisites

First, you'll need to install some tools that protobuf requires for the build process. On Ubuntu:

```bash
sudo apt-get install autoconf automake libtool curl make g++ unzip git
```

You'll also need to have an ARM GCC compiler toolchain installed on the host machine. On Ubuntu:

```bash
sudo apt-get install gcc-arm-linux-gnueabihf
```

Next, you need to clone the protobuf Github repo **and** download a copy of the `protoc` compiler for your host platform. For example, if you are building protobuf on a Linux machine, download `protoc` for Linux. 

In the example below, we will be building protobuf version 3.4.x on a Linux machine.

```bash
# Clone protobuf and switch to 3.4.x release branch
git clone https://github.com/google/protobuf
cd protobuf && git checkout 3.4.x

# Grab the 3.4.0 protobuf compiler for a Linux host and extract it to a new dir
wget https://github.com/google/protobuf/releases/download/v3.4.0/protoc-3.4.0-linux-x86_64.zip
mkdir protoc-3.4.0
unzip protoc-3.4.0-linux-x86_64.zip -d protoc-3.4.0/
```

Now we should be ready to build protobuf!

## Build

Let's configure the build based on our target platform i.e. 32-bit ARM on the Zynq. We specify the target platform (`arm-linux`), the cross-compilers for both C and C++, and finally point the build to a copy of `protoc` that runs on the host platform.

The full configuration command is as follows:

```bash
./configure --host=arm-linux CC=arm-linux-gnueabihf-gcc CXX=arm-linux-gnueabihf-g++ --with-protoc=./protoc-3.4.0/bin/protoc
```

With the build configured, we now just need to run:

```bash
make
make check
```

Once the build process is complete, both static and shared libraries for ARM 32-bit can be found under `src/.libs`; look for `libprotobuf.a` (static library) and `libprotobuf.so` (shared library).
