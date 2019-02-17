# Running Alpine in a chroot on the Lima Ultra

## Prerequisites

You need a [rooted Lima Ultra](../root/howto-root-ultra.md).

## Sources

The following sources have been used to write this.

- [Installing Alpine Linux in a chroot](https://wiki.alpinelinux.org/wiki/Installing_Alpine_Linux_in_a_chroot)
- [OpenRC Chroot support (Gentoo)](https://wiki.gentoo.org/wiki/OpenRC#Chroot_support)

## Goal

This installs Alpine Linux in a chroot. This lets you install packages not available on the Lima, compile code, run services, etc.

## Mount the user partition

The Lima Ultra root partition is only 2 GB, but there is a 3 GB user data partition which can be used for extra space.

We won't add it to fstab in this document, instead we will mount it manually. Feel free to do otherwise.

    mkdir /user
    mount /dev/disk/by-partlabel/user /user

Now let's create the root directory for the distro.

    mkdir /user/alpine

## Setup a few variables

You should [adapt the mirror URL](http://dl-cdn.alpinelinux.org/alpine/MIRRORS.txt) depending on your location.

    export chroot_dir="/user/alpine"
    export mirror="http://alpine.42.fr"
    export branch="v3.9"

## Bootstrap the chroot contents

    wget "${mirror}/${branch}/main/armv7/apk-tools-static-2.10.3-r1.apk"
    tar -xzf apk-tools-static-*.apk
    ./sbin/apk.static \
        -X "${mirror}/${branch}/main" \
        -U --allow-untrusted --root "${chroot_dir}" \
        --initdb add alpine-base

    mkdir -p "${chroot_dir}/home/root"
    mkdir -p "${chroot_dir}/etc/apk"
    echo "${mirror}/${branch}/main" > "${chroot_dir}/etc/apk/repositories"

## Create helper scripts

We will store stuff in `/alpine`.

    mkdir /alpine

In that directory, create the file `conf.sh`...

    mountpoint="/user"
    chroot_dir="$mountpoint/alpine"

... another file called `run`...

    #!/bin/bash

    cd "$(dirname "$0")"
    . conf.sh

    mount /dev/disk/by-partlabel/user "$mountpoint"

    cp /etc/resolv.conf "$chroot_dir/etc/resolv.conf"

    mount -t proc none "$chroot_dir/proc"
    mount -o bind /sys "$chroot_dir/sys"
    mount -o bind /dev "$chroot_dir/dev"

    chroot "$chroot_dir" openrc

... and finally `shell`.

    #!/bin/bash

    cd "$(dirname "$0")"
    . conf.sh

    chroot "${chroot_dir}" /bin/sh -l

Make `run` and `shell` executable.

    chmod +x /alpine/run
    chmod +x /alpine/shell

## Make it run on boot

Create executable file `/etc/init.d/run-alpine`:

    #!/bin/sh

    case "$1" in
        start)
            /home/root/alpine/run
        ;;
        *)
            echo "Usage: /etc/init.d/run-alpine start"
            exit 1
    esac
    exit 0

Add it to services on boot:

    update-rc.d run-alpine defaults

## Jump in the Alpine shell and finish configuration

Open the Alpine shell:

    /alpine/shell

Add useful packages:

    apk add alpine-sdk
    apk add nano

To make openrc run in a chroot, create an empty softlevel...

    touch /run/openrc/softlevel

... and configure your `rc.conf` like this:

    rc_sys="prefix"
    rc_controller_cgroups="NO"
    rc_depend_strict="NO"
    rc_need="!net !dev !udev-mount !sysfs !checkfs !fsck !netmount !logger !clock !modules"

## Add services

Configuration is now complete, but let's add a service to test everything works:


    apk add nginx
    rc-update add nginx

## Reboot

You can now exit the Alpine shell and reboot the Lima.

After boot, nginx should listen on port 80.
