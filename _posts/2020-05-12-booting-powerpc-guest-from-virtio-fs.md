# How to boot a PowerPC KVM guest from virtio-fs

## Introduction:
Virtio-fs is a shared file system that lets virtual machines access a directory tree on the host. Unlike existing approaches, it is designed to offer local file system semantics and performance.
Virtio-fs was started at Red Hat and is being developed in the Linux, QEMU, FUSE, and Kata Containers communities that are affected by code changes. More details on virtio-fs can be found [here](https://virtio-fs.gitlab.io/).

*This blog gives a step by step guide on how to setup an PowerPC KVM Environment to boot a KVM guest from virtio-fs as rootfs.*


## Environment used:

* HW: [MiHawk 2U2S Power9](https://openpowerfoundation.org/?resource_lib=wistron-corp-p93d2-2p-mihawk)
* Host OS: Fedora 31
* Kernel: 5.7.0-rc5-g752f08972

## Steps:

### 1. Create a working directory on host.

* `mkdir -p /home/virtio-fs`
* `cd /home/virtio-fs`
> For rest of the blog this folder is referred as $PWD

--------------

### 2. Needs a Guest kernel that supports virtio-fs as bootdisk, let's build one.

*Get kernel:*
* `git clone https://github.com/sathnaga/linux -b virtio-fs`

    > Mainline kernel has support for virtio-fs already, my above branch has
    > two additional patches to get rootfs feature and kernel configs
    > to enable it for my defconfig used below:
    >
    >    * Patch from https://patchwork.kernel.org/patch/11134865/mbox to get virtio-fs as rootfs support.
    >
    >    * Patch to enable virtio fs kernel configs to support on guest kernel.


*Compile kernel:*
* `cd linux`
* `make ppc64le_guest_defconfig`
* `make -j 120 -s`
* `cd ..`

Now we have guest kernel at **$PWD/linux/vmlinux**

--------------

### 3. Needs a qemu that supports virtio-fs, let's build one.

*Get Qemu:*
* `git clone https://github.com/qemu/qemu.git`
> this blog used code from master branch commit: [debe78ce14bf8f8940c2bdf3ef387505e9e035a9](https://github.com/qemu/qemu/commit/debe78ce14bf8f8940c2bdf3ef387505e9e035a9)



*Compile Qemu:*
* `cd qemu`
* `./configure --target-list=ppc64-softmmu --enable-debug --disable-werrormake`
* `make -j 120 -s`
* `cd ..`

Now we have qemu binary at **$PWD/qemu/ppc64-softmmu/qemu-system-ppc64**

--------------

### 4. Needs a virtiofs daemon, let's build one.

* `cd qemu`
* `make -j 8 virtiofsd`
* `cd ..`

Now we have virtio fs daemon binary at **$PWD/qemu/virtiofsd**

------------

### 5. Let's build a Fedora root file system based on F32:
* `mkdir $PWD/virtio-fs-root`
* `dnf --installroot=$PWD/virtio-fs-root --releasever=32 install system-release vim-minimal systemd passwd dnf rootfiles pciutils`

    >    Select yes to install packages.
    >
    >    Incase to install additional packages, use below commandline,
    >
    > dnf --installroot=$PWD/virtio-fs-root --releasever=32 install "packages to install"

* `openssl passwd -1 123456`

    `$1$sWUb0J.r$cyjCLYRfstQ0xEHVZ45UZ/`

    > let's create a passwd for guest root file system, in above command `123456` is our guest password and output generated is md5 password of same, change the password as per your need and use the generated one in shadow file.
* Edit password field of $PWD/virtio-fs-root/etc/shadow with above md5sum password.
    > Before edit:
    >
    > $ `head -1 $PWD/virtio-fs-root/etc/shadow`
    >
    > root:*:18292:0:99999:7:::
    >
    > After edit:
    >
    > $ `head -1 $PWD/virtio-fs-root/etc/shadow`
    > root:$1$sWUb0J.r$cyjCLYRfstQ0xEHVZ45UZ/:18292:0:99999:7:::
*  `mknod $PWD/virtio-fs-root/dev/hvc0 c 1 2`
    > This creates serial device for guest console

* `cp $PWD/virtio-fs-root/usr/lib/systemd/system/serial-getty@.service  $PWD/virtio-fs-root/etc/systemd/system/getty.target.wants/serial-getty@hvc0.service`
* `cd $PWD/virtio-fs-root/etc/systemd/system/getty.target.wants/`
* `ln -s serial-getty@hvc0.service getty@hvc0.service`
* Edit getty@hvc0.service with below changes
    ```
    ...
    [Unit]
    ....
    ConditionPathExists=/dev/hvc0
    ....
    [Install]
    WantedBy=getty.target
    DefaultInstance=hvc0
    ```
> this is needed to get the guest serial console working fine.

*  `cd /home/virtio-fs`

--------------

### 6. Let's Boot PowerPC KVM guest using qemu cmdline with virtio-fs.

*  `$PWD/qemu/virtiofsd -o --socket-path=/tmp/vhostqemu -o source=$PWD/virtio-fs-root -o cache=none &`
> this command starts the virtio fs daemon in background

* Run below qemu command to start a KVM guest
```bash
$PWD/qemu/ppc64-softmmu/qemu-system-ppc64 \
        -m 4096 -object memory-backend-file,id=mem,size=4G,mem-path=/dev/shm,share=on -numa node,memdev=mem \
        -smp 4 -enable-kvm -serial mon:stdio -vga none -nographic \
        -kernel $PWD/linux/vmlinux \
        -append "rootfstype=virtiofs root=myfs rw" \
        -chardev socket,id=char0,path=/tmp/vhostqemu \
        -device vhost-user-fs-pci,queue-size=1024,chardev=char0,tag=myfs
```

> Login using the password created in previous steps, this blog used `123456` as guest root password, let's use
that to login, below are command outputs inside kvm guest.

```bash
localhost login:
localhost login: root
Password:
Last login: Tue May 12 14:30:59 on hvc0
[   16.610692] id (134) used greatest stack depth: 10736 bytes left
[root@localhost ~]#
# lscpu
Architecture:                    ppc64le
Byte Order:                      Little Endian
CPU(s):                          4
On-line CPU(s) list:             0-3
Thread(s) per core:              1
Core(s) per socket:              1
Socket(s):                       4
NUMA node(s):                    1
Model:                           2.3 (pvr 004e 1203)
Model name:                      POWER9 (architected), altivec supported
Hypervisor vendor:               KVM
Virtualization type:             para
L1d cache:                       128 KiB
L1i cache:                       128 KiB
NUMA node0 CPU(s):               0-3
Vulnerability Itlb multihit:     Not affected
Vulnerability L1tf:              Mitigation; RFI Flush, L1D private per thread
Vulnerability Mds:               Not affected
Vulnerability Meltdown:          Mitigation; RFI Flush, L1D private per thread
Vulnerability Spec store bypass: Mitigation; Kernel entry/exit barrier (eieio)
Vulnerability Spectre v1:        Mitigation; __user pointer sanitization, ori31
                                 speculation barrier enabled
Vulnerability Spectre v2:        Mitigation; Software count cache flush, Softwar
                                 e link stack flush
Vulnerability Tsx async abort:   Not affected

[root@localhost ~]# cat /etc/os-release
NAME=Fedora
VERSION="32 (Thirty Two)"
ID=fedora
VERSION_ID=32
VERSION_CODENAME=""
PLATFORM_ID="platform:f32"
PRETTY_NAME="Fedora 32 (Thirty Two)"
ANSI_COLOR="0;34"
LOGO=fedora-logo-icon
CPE_NAME="cpe:/o:fedoraproject:fedora:32"
HOME_URL="https://fedoraproject.org/"
DOCUMENTATION_URL="https://docs.fedoraproject.org/en-US/fedora/f32/system-administrators-guide/"
SUPPORT_URL="https://fedoraproject.org/wiki/Communicating_and_getting_help"
BUG_REPORT_URL="https://bugzilla.redhat.com/"
REDHAT_BUGZILLA_PRODUCT="Fedora"
REDHAT_BUGZILLA_PRODUCT_VERSION=32
REDHAT_SUPPORT_PRODUCT="Fedora"
REDHAT_SUPPORT_PRODUCT_VERSION=32
PRIVACY_POLICY_URL="https://fedoraproject.org/wiki/Legal:PrivacyPolicy"
[root@localhost ~]# uname -a
Linux localhost 5.7.0-rc5-g7aa8a99a5-dirty #5 SMP Tue May 12 07:33:17 EDT 2020 ppc64le ppc64le ppc64le GNU/Linux
[root@localhost ~]# lspci
00:00.0 Mass storage controller: Red Hat, Inc. Device 105a (rev 01)
```


> _**Now We have PowerPC KVM guest booted with virtio-fs as bootdisk.**_


This blog is written based on below reference documents on virtio-fs and my additional steps that are needed for setup on PowerPC environment.
* [https://virtio-fs.gitlab.io/howto-qemu.html](https://virtio-fs.gitlab.io/howto-qemu.html)
* [https://virtio-fs.gitlab.io/howto-boot.html](https://virtio-fs.gitlab.io/howto-qemu.html)

Thanks for reading, Hope this blog helps you in someways :-)

<script src="https://utteranc.es/client.js"
       repo="sathnaga/sathnaga.github.io"
       issue-term="url"
       theme="github-light"
       crossorigin="anonymous"
       async>
</script>
