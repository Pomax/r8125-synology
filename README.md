# r8125-synology

This repo contains code and instructions to compile and install the RTL8125 driver for Synology DSM.

My current setup:

- Synology RS1219+ (avoton)
- DSM 7.2 (Linux 3.10.108)
- TRENDnet 2.5GBASE-T PCIe Network Adapter (TEG-25GECTX)

File an issue if you run into problems on other Synology DSM versions and platforms.

## Compile module

Steps below are based [this](https://help.synology.com/developer-guide/getting_started/prepare_environment.html) and [this](https://help.synology.com/developer-guide/compile_applications/compile_open_source_projects.html) docs.

1. Create a suitable Linux environment (not your NAS!). I used a Ubuntu 22.04 LTS Docker image.
```bash
$ docker run --privileged -it ubuntu:22.04
```

2. Set up environment:

```bash
$ apt-get update
$ apt-get install git python3 python3-pip
$ mkdir -p /toolkit
$ cd /toolkit
$ git clone https://github.com/SynologyOpenSource/pkgscripts-ng
```

3. Deploy chroot environment:

```bash
$ cd /toolkit/pkgscripts-ng
$ git checkout DSM7.2
$ ./EnvDeploy -v 7.2 -p avoton # replace 'avoton' with your platform, listed as "package arch" for your Synology model on https://kb.synology.com/en-uk/DSM/tutorial/What_kind_of_CPU_does_my_NAS_have
$ cp /etc/ssl/certs/ca-certificates.crt /toolkit/build_env/ds.avoton-7.2/etc/ssl/certs/ca-certificates.crt # replace 'ds.avoton-7.2' with your platform and version
```

4. Chroot into environment:

```bash
$ chroot /toolkit/build_env/ds.avoton-7.2
```

5. Download code:

```bash
$ mkdir -p /usr/src
$ cd /usr/src
$ git clone https://github.com/tabrezm/r8125-synology
$ cd r8125-synology/src
```

If you're compiling for a different DSM version, you need to update `BASEDIR` in the makefile:

```
	BASEDIR := /usr/local/x86_64-pc-linux-gnu/x86_64-pc-linux-gnu/sys-root/usr/lib/modules/DSM-7.2
```

6. Compile module:

```bash
$ make
```

You should have no build errors and `r8215.ko` in the current directory.

## Install module

1. Copy `r8215.ko` to a location available from your Synology device, like a shared folder or using SCP.

2. Connect using SSH and run the following commands:

```bash
$ sudo -i
$ cp r8215.ko /lib/modules
$ insmod /lib/modules/r8125.ko
```

3. Confirm the module is loaded:

```
root@synology:~# lspci -v | grep r8125
	Kernel driver in use: r8125
```

4. Set up new interface:

```bash
$ ip link set up eth4 # replace 'eth4' with your interface
```

At this point, you should be able to see interface details using `ifconfig eth4`.

5. Update modules so `r8125` is loaded automatically at next boot:

```bash
$ ln -s /bin/kmod /sbin/depmod
$ depmod -a # warnings are safe to ignore
```

---

## Change log

- 2023-07-08: Update driver version to 9.009.02.
- 2023-07-08: Initial commit, support for driver version 9.006.04.
