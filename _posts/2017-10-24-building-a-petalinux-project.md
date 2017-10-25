---
layout: post
title: "Beginner's Guide: Build a PetaLinux Project for Zynq"
author: {{ site.author.name }}
---

## Introduction

PetaLinux is the default build system provided by Xilinx for its SoC platforms (Zynq and MicroBlaze-based). In a nutshell, PetaLinux provides a set of scripts that run on top of the [Yocto Project](https://www.yoctoproject.org/) embedded Linux build system. PetaLinux is therefore simpler to use, but less configurable in many regards; this is not a problem for a beginner to the platform, such as myself.

In this guide, I will show you how to:

1. Install PetaLinux
2. Create a new sample project targeting the Zynq platform
3. Build the project for SD card boot

Note that this guide assumes that you are using Ubuntu 16.04 LTS with at least 50 GB of free space available on your hard disk.

## Installing PetaLinux

### Prerequisites

On Ubuntu 16.04 LTS, you will need to run the following command to grab PetaLinux's dependencies:

```bash
sudo apt-get install libssl-dev ncurses-dev zlib1g-dev zlib1g:i386 lib32ncursesw5 lib32gomp1 gcc gawk tftpd git chr-path autoconf libtool socat texinfo gcc-multilib libsdl1.2-dev libglib2.0-dev xvfb
```

### Download and Install

The PetaLinux installation process is fairly straightforward.

Let's begin by grabbing the latest PetaLinux installer from the [Xilinx PetaLinux downloads](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-design-tools.html) page. As of the creation of this guide, the latest version of PetaLinux available for download is v2017.3. The installer is quite large in size, so it may take some time to download. Also, you will need to create a free Xilinx account to access the download link.

Once the download is complete, navigate to the directory and run the following to install the PetaLinux toolchain to `~/petalinux`.

```bash
mkdir ~/petalinux
chmod +x petalinux-v2017.3-final-installer.run
./petalinux-v2017.3-final-installer.run ~/tools/petalinux
```

You will be prompted to accept some licenses; just type `q` followed by `y` at each prompt.

Finally, let's create an alias for sourcing the PetaLinux settings script, as you will need to do this whenever you open a new shell. You can place this line in your `~/.bash_profile`, or even just have it execute by just placing the source statement and destination directly.

```bash
alias peta='source ~/tools/petalinux/petalinux-v2017.3-final/setttings.sh`
```

If you would like to disable WebTalk (usage reporting), run: `petalinux-util --webtalk off`.

## Create a New Zynq Project

Now that PetaLinux is installed, let's create a new PetaLinux project for the Zynq.

First, make sure that the PetaLinux tools are in your `PATH` by running the alias we defined above:

```bash
peta
```

Let's create a blank project called `hello-zynq` under the directory `~/petalinux-projects`:

```bash
cd ~/petalinux-projects
petalinux-create -t project -n hello-zynq --template zynq
cd hello-zynq
```

If you run `ls`, you'll notice that there isn't much inside the project directory. This is because we have yet to run our first build! Remember the PetaLinux settings script we sourced in? Well, that script contains the locations of all the build tools, dependencies, and sources that our project will use during the build process.

## Configure and Build

The final step to get a working build is to point the project to the location of your HDF (Hardware Description File). This is the file you get from Vivado once you test and synthesize your FPGA design (*File > Export > Export Hardware...*). 

If you do not have an FPGA design, just create one of the example projects, synthesize it and generate the bitstream, and finally export it as an HDF for use with PetaLinux. Another option is to use the Zynq or Zedboard BSPs on the Downloads page linked above. 

Once you have an HDF (assuming it is in the project root):

```bash
petalinux-config --get-hw-description=.
```

Exit the menuconfig by hitting `<ESC>` twice in a row.

The project should now be ready to build. 

```bash
petalinux-build
```

This will take a **lot of time**, so grab a cup of coffee.

The command above will:

1. Pull in all required sources from remote repos, if necessary
2. Build FSBL, U-Boot, DTC, and the Linux kernel from source
3. Compile the device tree DTC to a binary DTB
4. Build the rootfs and compress it

## Build Output

The resulting files will be located under the directory `<project_root>/images/linux`.

For SD card boot, the important files are `image.ub` (Linux kernel + device tree), `rootfs.tar.gz` (BusyBox-based root filesystem), and `BOOT.bin` (FSBL + bitstream + U-Boot).

Notice that there is no `BOOT.bin` by default. To create the `BOOT.bin` file, run:

```bash
petalinux-package --boot --format BIN --fsbl images/linux/zynq_fsbl.elf --kernel images/linux/image.ub --u-boot images/linux/u-boot.elf --fpga images/linux/<NAME_OF_BITSREAM>.bit
```

If an error pops up regarding the absence of the `bootgen` utility, you might have to install the Xilinx SDK and either source in the settings script or use the SDK's GUI.
