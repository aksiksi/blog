---
layout: post
title: "Fixing pulseaudio issues with QEMU/libvirt on Arch"
author: {{ site.author.name }}
tags:
    - arch
    - qemu
    - libvirt
    - pulseaudio
---

# Check the QEMU log

If you're having audio issues with pulseaudio with QEMU on Arch, start by checking the QEMU log:

```
$ sudo tail -f /var/log/libvirt/qemu/win10.log
-device hda-micro,audiodev=hda \
-audiodev pa,id=hda,server=unix:/tmp/pulse-socket \
-sandbox on,obsolete=deny,elevateprivileges=deny,spawn=deny,resourcecontrol=deny \
-msg timestamp=on
2020-07-09 02:55:21.277+0000: Domain id=1 is tainted: custom-argv
char device redirected to /dev/pts/0 (label charserial0)
pulseaudio: pa_context_connect() failed
pulseaudio: Reason: Connection refused
pulseaudio: Failed to initialize PA contextaudio: Could not init `pa' audio driver
audio: warning: Using timer based audio emulation
```

In this case, QEMU is complaining that it cannot connect to the pulseaudio server.

# Ensure that pulseaudio is running for your user

I do this in two steps. First, make sure that the server is running for your user. Here, I want passthrough for the `work` user:

```
$ ps aux | grep pulseaudio
aksiksi     1346  0.2  0.0 1504860 7552 ?        S<sl Jun25  47:47 /usr/bin/pulseaudio --daemonize=no
work      534916  3.8  0.0 2291876 15640 ?       S<sl 23:15   0:31 /usr/bin/pulseaudio --daemonize=no
```

Second, use `pactl` to ensure that you can connect to the server:

```
$ pactl info
Server String: /run/user/1002/pulse/native
Library Protocol Version: 33
Server Protocol Version: 33
Is Local: yes
Client Index: 16
Tile Size: 65472
User Name: work
```

This is what you'd get if the server is not accessible:

```
$ pactl info
Connection failure: Connection refused
pa_context_connect() failed: Connection refused
```

# Verify QEMU user and group

You need to make sure that you've set a QEMU `user` and `group` that matches the user you want pulseaudio passthrough to work with.

```
$ sudo vim /etc/libvirt/qemu.conf
...
user = "work"
group = "kvm"
```

You can confirm that your user is in a group like so:

```
$ groups
sys network scanner power libvirt docker video optical lp kvm input audio wheel work dev
```

# Verify pulseaudio socket user

Next, head over to your libvirt config file and verify the user ID used for the pulseaudio socket:

```
$ sudo EDITOR=vim virsh edit win10
...
    <qemu:arg value='-device'/>
    <qemu:arg value='ich9-intel-hda,bus=pcie.0,addr=0x1b'/>
    <qemu:arg value='-device'/>
    <qemu:arg value='hda-micro,audiodev=hda'/>
    <qemu:arg value='-audiodev'/>
    <qemu:arg value='pa,id=hda,server=/run/user/1002/pulse/native'/>
```

Note: this should match the socket path from `pactl info` above.

# Restart pulseaudio server

At this point, everything should be good. If you still are hitting the error above in your QEMU logs, it's time to restart QEMU:

```
$ pulseaudio --check
$ pulseaudio --kill
$ pulseaudio --start
```
