---
title: "Building VyOS using Podman"
date: 2023-04-26
draft: false
tags:
  - linux
  - networking
---

The VyOS developers only provide free access to nightly builds of VyOS, and not
stable releases. If you want to run a stable build of VyOS, you will need to
build it yourself.

This will work just fine on any modern Linux distribution--even ones running
inside of WSL2 on Windows 11.

## Build steps

1. Install podman

```bash
sudo apt install podman
```

2. Pull the build environment w/ podman

```bash
sudo podman pull docker.io/vyos/vyos-build:equuleus
```

3. Fetch source code from github

```bash
git clone -b equuleus --single-branch https://github.com/vyos/vyos-build
```

4. Enter build environment

```bash
cd vyos-build
sudo podman run --rm -it --privileged -v $(pwd):/vyos -w /vyos vyos/vyos-build:equuleus bash
```

5. Build it

```bash
./configure --architecture amd64 --build-by "hello@example.com"
sudo make iso
```

Once finished, exit the container. Inside the "build" directory will be the
.iso image you'll need to boot in your hypervisor.
