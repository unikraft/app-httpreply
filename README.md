# Httpreply on Unikraft

This application starts an Httpreply web server with Unikraft.
Follow the instructions below to set up, configure, build and run Httpreply.

## Quick Setup (aka TLDR)

For a quick setup, run the commands below.
Note that you still need to install the [requirements](#requirements).

For building and running everything for `x86_64`, follow the steps below:

```console
git clone https://github.com/unikraft/app-httpreply httpreply
cd httpreply/
./scripts/setup.sh
wget https://raw.githubusercontent.com/unikraft/app-testing/staging/scripts/generate.py -O scripts/generate.py
chmod a+x scripts/generate.py
./scripts/generate.py
./scripts/build/make-qemu-x86_64.sh
./scripts/run/qemu-x86_64.sh
```

This will configure, build and run the `httpreply` application.
You can see how to test it in the [running section](#run).

The same can be done for `AArch64`, by running the commands below:

```console
git clone https://github.com/unikraft/app-httpreply httpreply
cd httpreply/
./scripts/setup.sh
wget https://raw.githubusercontent.com/unikraft/app-testing/staging/scripts/generate.py -O scripts/generate.py
chmod a+x scripts/generate.py
./scripts/generate.py
./scripts/build/make-qemu-arm64.sh
./scripts/run/qemu-arm64.sh
```

Similar to the `x86_64` build, this will start the `httpreply` server.
Information about every step is detailed below.

## Requirements

In order to set up, configure, build and run Httpreply on Unikraft, the following packages are required:

* `build-essential` / `base-devel` / `@development-tools` (the meta-package that includes `make`, `gcc` and other development-related packages)
* `sudo`
* `flex`
* `bison`
* `git`
* `wget`
* `uuid-runtime`
* `qemu-system-x86`
* `qemu-system-arm`
* `qemu-kvm`
* `sgabios`
* `gcc-aarch64-linux-gnu`

GCC >= 8 is required to build Httpreply on Unikraft.

On Ubuntu/Debian or other `apt`-based distributions, run the following command to install the requirements:

```console
sudo apt install -y --no-install-recommends \
  build-essential \
  sudo \
  gcc-aarch64-linux-gnu \
  libncurses-dev \
  libyaml-dev \
  flex \
  bison \
  git \
  wget \
  uuid-runtime \
  qemu-kvm \
  qemu-system-x86 \
  qemu-system-arm \
  sgabios
```

Running Httpreply Unikraft with QEMU requires networking support.
For this to work properly a specific configuration must be enabled for QEMU.
Run the commands below to enable that configuration (for the network bridge to work):

```console
sudo mkdir /etc/qemu/
echo "allow all" | sudo tee /etc/qemu/bridge.conf
```

## Set Up

The following repositories are required for Httpreply:

* The application repository (this repository): [`app-httpreply`](https://github.com/unikraft/app-httpreply)
* The Unikraft core repository: [`unikraft`](https://github.com/unikraft/unikraft)
* Library repositories:
  * The networking stack library: [`lib-lwip`](https://github.com/unikraft/lib-lwip)

Follow the steps below for the setup:

  1. First clone the [`app-httpreply` repository](https://github.com/unikraft/app-httpreply) in the `httpreply/` directory:

     ```console
     git clone https://github.com/unikraft/app-httpreply httpreply
     ```

     Enter the `httpreply/` directory:

     ```console
     cd httpreply/

     ls -F
     ```

     This will print the contents of the repository:

     ```text
     [...] README.md  defconfigs/  kraft.cloud.yaml  kraft.yaml  main.c  scripts/
     ```

  1. While inside the `httpreply/` directory, clone all required repositories by using the `setup.sh` script:

     ```console
     ./scripts/setup.sh
     ```

  1. Use the `tree` command to inspect the contents of the `workdir/` directory:

     ```console
     tree -F -L 2 workdir/
     ```

     The layout of the `workdir/` directory should look something like this:

     ```text
     workdir/
     |-- libs/
     |   `-- lwip/
     `-- unikraft/
         |-- arch/
         |-- Config.uk
         |-- CONTRIBUTING.md
         |-- COPYING.md
         |-- include/
         |-- lib/
         |-- Makefile
         |-- Makefile.uk
         |-- plat/
         |-- README.md
         |-- support/
         `-- version.mk

     10 directories, 7 files
     ```

## Scripted Building and Running

To make it easier to build, run and test different configurations, the repository provides a set of scripts that do everything required.
These are scripts used for building different configurations of the httpreply server and for running these with all the requirements behind the scenes: creating network configurations, setting up archives etc.

First of all, grab the [`generate.py` script](https://github.com/unikraft/app-testing/blob/staging/scripts/generate.py) and place it in the `scripts/` directory by running:

```console
wget https://raw.githubusercontent.com/unikraft/app-testing/staging/scripts/generate.py -O scripts/generate.py
chmod a+x scripts/generate.py
```

Now, run the `generate.py` script.
You must run it from the root directory of this repository:

```console
./scripts/generate.py
```

The scripts (as shell scripts) are now generated in `scripts/build/` and `scripts/run/`:

```text
scripts//
|-- build/
|   |-- kraft-fc-arm64.sh*
|   |-- kraft-fc-x86_64.sh*
|   |-- kraft-qemu-arm64.sh*
|   |-- kraft-qemu-x86_64.sh*
|   |-- make-fc-arm64.sh*
|   |-- make-fc-x86_64.sh*
|   |-- make-qemu-arm64.sh*
|   `-- make-qemu-x86_64.sh*
|-- generate.py*
|-- run/
|   |-- fc-arm64.json
|   |-- fc-arm64.sh*
|   |-- fc-x86_64.json
|   |-- fc-x86_64.sh*
|   |-- kraft-fc-arm64.sh*
|   |-- kraft-fc-x86_64.sh*
|   |-- kraft-qemu-arm64.sh*
|   |-- kraft-qemu-x86_64.sh*
|   |-- qemu-arm64.sh*
|   `-- qemu-x86_64.sh*
|-- run.yaml
`-- setup.sh*
```

They are shell scripts, so you can use an editor or a text viewer to check their contents:

```console
cat scripts/run/fc-x86_64.sh
```

Now, invoke each script to build and run the application.
A sample build and run set of commands is:

```console
./scripts/build/make-qemu-x86_64.sh
./scripts/run/qemu-x86_64.sh
```

Note that Firecracker only works with initrd (not 9pfs).
And Firecracker networking is not yet upstream.

## Detailed Steps

### Configure

Configuring, building and running a Unikraft application depends on our choice of platform and architecture.
Currently, supported platforms are QEMU (KVM), Xen and linuxu.
QEMU (KVM) is known to be working, so we focus on that.

Supported architectures are x86_64 and AArch64.

Use the corresponding the configuration files (`defconfigs/*`), according to your choice of platform and architecture.

### QEMU x86_64

Use the `defconfigs/qemu-x86_64` configuration file together with `make defconfig` to create the configuration file:

```console
UK_DEFCONFIG=$(pwd)/defconfigs/qemu-x86_64 make defconfig
```

This results in the creation of the `.config` file:

```console
ls .config
.config
```

The `.config` file will be used in the build step.

### QEMU AArch64

Use the `defconfigs/qemu-arm64` configuration file together with `make defconfig` to create the configuration file:

```console
UK_DEFCONFIG=$(pwd)/defconfigs/qemu-arm64 make defconfig
```

Similar to the x86_64 configuration, this results in the creation of the `.config` file that will be used in the build step.

## Build

Building uses as input the `.config` file from above, and results in a unikernel image as output.
The unikernel output image, together with intermediary build files, are stored in the `build/` directory.

### Clean Up

Before starting a build on a different platform or architecture, you must clean up the build output.
This may also be required in case of a new configuration.

Cleaning up is done with 3 possible commands:

* `make clean`: cleans all actual build output files (binary files, including the unikernel image)
* `make properclean`: removes the entire `build/` directory
* `make distclean`: removes the entire `build/` directory **and** the `.config` file

Typically, you would use `make properclean` to remove all build artifacts, but keep the configuration file.

### QEMU x86_64

Building for QEMU x86_64 assumes you did the QEMU x86_64 configuration step above.
Build the Unikraft Httpreply image for QEMU x86_64 by using the command below:

```console
make -j $(nproc)
```

This will print a list of files that are generated by the build system.

```text
[...]
  LD      httpreply_qemu-x86_64.dbg
  UKBI    httpreply_qemu-x86_64.dbg.bootinfo
  SCSTRIP httpreply_qemu-x86_64
  GZ      httpreply_qemu-x86_64.gz
make[1]: Leaving directory 'httpreply/workdir/unikraft'
```

At the end of the build command, the `httpreply-x86_64` unikernel image is generated.
This image is to be used in the run step.

### QEMU AArch64

If you had configured and build a unikernel image for another platform or architecture (such as x86_64) before, then:

1. Do a cleanup step with `make properclean`.

1. Configure for QEMU AAarch64, as shown above.

1. Follow the instructions below to build for QEMU AArch64.

Building for QEMU AArch64 assumes you did the QEMU AArch64 configuration step above.
Build the Unikraft Httpreply image for QEMU AArch64 by using the same command as for x86_64:

```console
make -j $(nproc)
```

This will print a list of files that are generated by the build system.

```text
[...]
  LD      httpreply_qemu-arm64.dbg
  UKBI    httpreply_qemu-arm64.dbg.bootinfo
  SCSTRIP httpreply_qemu-arm64
  GZ      httpreply_qemu-arm64.gz
make[1]: Leaving directory 'httpreply/workdir/unikraft'
```

Similarly to x86_64, at the end of the build command, the `httpreply-arm64` unikernel image is generated.
This image is to be used in the run step.

## Run

The resulting image can be run with the `qemu-system-*` commands.
In order to run `httpreply` you need to first set up a network bridge.
All this is part of the `./run-qemu-*.sh` scripts.

### QEMU x86_64

To run the QEMU x86_64 build, use `./run-qemu-x86_64.sh`:

```console
sudo ip link set dev tap0 down 2> /dev/null
sudo ip link del dev tap0 2> /dev/null
sudo ip link set dev virbr0 down 2> /dev/null
sudo ip link del dev virbr0 2> /dev/null
sudo qemu-system-x86_64 \
    -kernel workdir/build/httpreply_qemu-x86_64 \
    -nographic \
    -m 64M \
    -netdev bridge,id=en0,br=virbr0 -device virtio-net-pci,netdev=en0 \
    -append "netdev.ipv4_addr=172.44.0.2 netdev.ipv4_gw_addr=172.44.0.1 netdev.ipv4_subnet_mask=255.255.\
    -cpu max
```

This will start the `httpreply` application:

```text
1: Set IPv4 address 172.44.0.2 mask 255.255.255.0 gw 172.44.0.1
en1: Added
en1: Interface is up
Powered by
o.   .o       _ _               __ _
Oo   Oo  ___ (_) | __ __  __ _ ' _) :_
oO   oO ' _ `| | |/ /  _)' _` | |_|  _)
oOo oOO| | | | |   (| | | (_) |  _) :_
 OoOoO ._, ._:_:_,\_._,  .__,_:_, \___)
                  Atlas 0.13.1~5eb820bd
Listening on port 8123...
```

The server listens for connections on the `172.44.0.2` address advertised.
A web client (such as `wget`) is required to query the server.

Open another console and use the `wget` command below to query the server:

```console
wget 172.44.0.2:8123
```

This will download [the `html` response](https://github.com/unikraft/app-httpreply/blob/staging/main.c#L42-L50) that the application sends:

```text
--2023-07-03 08:38:20--  http://172.44.0.2:8123/
Connecting to 172.44.0.2:8123... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: ‘index.html’

index.html              [ <=>                ]     159  --.-KB/s    in 0s

2023-07-03 08:38:20 (10,9 MB/s) - ‘index.html’ saved [159]
```

To close the QEMU Httpreply server, use the `Ctrl+a x` keyboard shortcut;
that is press the `Ctrl` and `a` keys at the same time and then, separately, press the `x` key.

### QEMU AArch64

To run the AArch64 build, use `./run-qemu-aarch64.sh`:

```console
sudo ip link set dev tap0 down 2> /dev/null
sudo ip link del dev tap0 2> /dev/null
sudo ip link set dev virbr0 down 2> /dev/null
sudo ip link del dev virbr0 2> /dev/null
sudo qemu-system-aarch64 \
    -machine virt \
    -kernel workdir/build/httpreply_qemu-x86_64 \
    -nographic \
    -m 64M \
    -netdev bridge,id=en0,br=virbr0 -device virtio-net-pci,netdev=en0 \
    -append "netdev.ipv4_addr=172.44.0.2 netdev.ipv4_gw_addr=172.44.0.1 netdev.ipv4_subnet_mask=255.255.\
    -cpu max
```

Similar to running for x86_64, this will start the `httpreply` application:

```text
1: Set IPv4 address 172.44.0.2 mask 255.255.255.0 gw 172.44.0.1
en1: Added
en1: Interface is up
Powered by
o.   .o       _ _               __ _
Oo   Oo  ___ (_) | __ __  __ _ ' _) :_
oO   oO ' _ `| | |/ /  _)' _` | |_|  _)
oOo oOO| | | | |   (| | | (_) |  _) :_
 OoOoO ._, ._:_:_,\_._,  .__,_:_, \___)
                  Atlas 0.13.1~5eb820bd
Listening on port 8123...
```

Open another console and use the `wget` command, similar to the QEMU x86_64 run above:

```console
wget 172.44.0.2:8123
```

This will download [the `html` response](https://github.com/unikraft/app-httpreply/blob/staging/main.c#L42-L50) that the application sends:

```text
--2023-07-03 08:38:20--  http://172.44.0.2:8123/
Connecting to 172.44.0.2:8123... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: ‘index.html’

index.html              [ <=>                ]     159  --.-KB/s    in 0s

2023-07-03 08:38:20 (10,9 MB/s) - ‘index.html’ saved [159]
```

Similarly, to close the QEMU Httpreply server, use the `Ctrl+a x` keyboard shortcut.
