---
type: reference
title: "Modprobe Options"
created: 2026-06-21
updated: 2026-06-21
tags:
  - linux
  - kernel
  - modules
status: seed
related:
  - "[[NVIDIA GSP Firmware]]"
---

# Modprobe Options

Kernel modules accept **parameters** that change their behaviour at load time.
`modprobe` reads configuration from `/etc/modprobe.d/*.conf` to set those
parameters, blacklist modules, or alias them — persistently across boots.

## Setting a module parameter

```
# /etc/modprobe.d/<name>.conf
options <module> <param>=<value> [<param2>=<value2> ...]
```

Example (disabling NVIDIA GSP firmware — see [[NVIDIA GSP Firmware]]):

```
options nvidia NVreg_EnableGpuFirmware=0
```

> [!key-insight]
> A parameter only takes effect when the module **loads**. If the module is built
> into the initramfs (drivers needed early, like GPU or storage), you must
> **regenerate the initramfs** for the option to apply at boot — editing the conf
> alone isn't enough.

```bash
sudo mkinitcpio -P     # rebuild all initramfs images
```

For modules loaded later (not in initramfs), unloading + reloading or a reboot
suffices.

## Blacklisting a module

```
# /etc/modprobe.d/blacklist-foo.conf
blacklist nouveau          # prevent autoload
install nouveau /bin/false # hard-block even as a dependency
```

`blacklist` stops automatic loading; the `install ... /bin/false` line also blocks
it from being pulled in as another module's dependency (used e.g. when binding a
GPU to `vfio-pci` for [[VFIO GPU Passthrough]]).

## Inspecting

```bash
modinfo <module>                      # available parameters + descriptions
systool -m <module> -A                # current parameter values (sysfs)
cat /sys/module/<module>/parameters/<param>
lsmod                                 # loaded modules
```

## Verifying a parameter took

```bash
cat /sys/module/nvidia/parameters/NVreg_EnableGpuFirmware
```

## Related

- [[NVIDIA GSP Firmware]] — a concrete `options nvidia ...` use
- [[VFIO GPU Passthrough]] — blacklisting + vfio binding
