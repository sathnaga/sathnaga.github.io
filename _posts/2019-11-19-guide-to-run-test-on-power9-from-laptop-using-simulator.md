# Guide to run commands/tests on a IBM Power9 using systemsim(mambo) from your Laptop using op-test and buildroot

## Env Used:

* Intel(R) Core(TM) i5-5300U CPU @ 2.30GHz
* Fedora 31(5.3.7-301.fc31.x86_64)

## Pre-requisite:
Install packages needed for cross compilation

    For Ubuntu env:
        apt-get install gcc-powerpc64le-linux-gnu gcc valgrind \
        expect libssl-dev device-tree-compiler make \
        xz-utils
    For Fedora env:
        dnf install gcc-powerpc64le-linux-gnu \
        binutils-powerpc64-linux-gnu gcc make diffutils findutils \
        expect valgrind-devel dtc openssl-devel xz

## How to:

1. Clone [skiboot](https://github.com/open-power/skiboot#building), Cross compile  and get `skiboot.lid`.


2. Clone linux [kernel](https://github.com/torvalds/linux), Cross compile  and get `vmlinux`.
    ```
    ARCH=powerpc CROSS=powerpc64le-linux-gnu- make ppc64le_defconfig
    ARCH=powerpc CROSS=powerpc64le-linux-gnu- make -j `nproc`
    ```
3. Clone [buildroot](https://github.com/open-power/buildroot), Compile and get rootfs image.

    * run 'make menuconfig'.
    * Choose

        Target options  --->
             Target Architecture (PowerPC64 (little endian))
             Target Architecture Variant (power8)

        Filesystem images  --->
            cpio the root filesystem (for use as an initial RAM filesystem)`

        and other packages you would need.
    * run `make`.
    * Find root filesystem in output/images(`rootfs.cpio`).


4. Install systemsim(mambo) [rpm](ftp://public.dhe.ibm.com/software/server/powerfuncsim/p9/packages) for power9 and check for systemsim(mambo) binary for power9 under `/opt/ibm/systemsim-p9/run/p9/`.


5. Create op-test config to run [OpTestMamboBuildRoot](https://github.com/open-power/op-test/pull/545/commits/bbc9ffc90e12557db0a4a08433d4831e422bd57c) test, same test can modified to run different commands inside power9 host systemsim.
```
    $cat mambo_p9.cfg
    [op-test]
    bmc_type=mambo
    mambo_binary=/opt/ibm/systemsim-p9/run/p9/power9;5dc2cbf4
    flash_skiboot=/home/satheesh/github/skiboot/skiboot.lid
    flash_kernel=/home/satheesh/github/linux/vmlinux
    flash_initramfs=/home/satheesh/github/buildroot/output/rootfs.cpio
    host_user=root
    host_password=
```

6. Run [OpTestMamboBuildRoot](https://github.com/open-power/op-test/pull/545/commits/bbc9ffc90e12557db0a4a08433d4831e422bd57c) test, which executes `cat /proc/interrupts` and `cat /proc/cpuinfo` inside a power9 host systemsim(mambo) env as shown below.
```

    $./op-test -c mambo_p9.cfg --run testcases.OpTestMamboBuildRoot.OpTestMamboBuildRoot

    ...
    ...
    runTest (testcases.OpTestMamboBuildRoot.OpTestMamboBuildRoot) ... Using initial run script skiboot.tcl
    No network support selected^M
    INFO: 0: (0): !!!!!! Simulator now in TURBO mode !!!!!!
    [    0.000170223,5] OPAL v6.5-86-g1c282887 starting...^M
    [    0.000176398,7] initial console log level: memory 7, driver 5^M
    [    0.000181940,6] CPU: P9 generation processor (max 4 threads/core)^M
    [    0.000187332,7] CPU: Boot CPU PIR is 0x0000 PVR is 0x004e1200^M
    [    0.000194047,7] OPAL table: 0x30113630 .. 0x30113ba0, branch table: 0x30002000^M
    [    0.000202048,7] Assigning physical memory map table for nimbus^M
    [    0.000208137,7] FDT: Parsing fdt @0x1f00000^M
    [    0.001023665,5] Enabling Mambo console^M
    [    0.001027953,5] CHIP: Detected Mambo simulator^M
    [    0.001096322,5] CHIP: Chip ID 0000 type: P9N DD2.30^M
    [    0.001346958,5] PLAT: Detected Mambo platform^M
    [    0.001558571,5] CPU: All 1 processors called in...^M
    [    0.001600800,3] SBE: Master chip ID not found.^M
    [    0.001767201,3] NVRAM: Partition at offset 0x0 has incorrect 0 length^M
    [    0.001773195,3] NVRAM: Re-initializing (size: 0x00040000)^M
    [    0.001935098,5] STB: secure boot not supported^M
    [    0.001943298,5] STB: trusted boot not supported^M
    [    0.002078002,4] FLASH: Can't load resource id:0. No system flash found^M
    [    0.002095952,4] FLASH: Can't load resource id:1. No system flash found^M

    ...
    ...
    [console-expect]#lsprop /sys/firmware/devicetree/base/ibm,firmware-versions
    lsprop /sys/firmware/devicetree/base/ibm,firmware-versions
    -sh: lsprop: not found^M^M
    [console-expect]#
    date
    which whoami && whoami

    [console-expect]#date
    which whoami && whoami
    Tue Nov 19 07:03:56 UTC 2019^M^M
    [console-expect]#/usr/bin/whoami^M^M
    root^M^M
    [console-expect]#echo $?
    echo $?
    0^M^M
    [console-expect]#stty -echo
    stty -echo
    [console-expect]#echo $?
    echo $?
    0^M^M
    [console-expect]#uname -a
    uname -a
    Linux buildroot 5.3.0-rc7 #1 SMP Wed Sep 4 14:34:53 IST 2019 ppc64le GNU/Linux^M^M
    [console-expect]#echo $?
    echo $?
    0^M^M
    [console-expect]#cat /proc/cpuinfo
    cat /proc/cpuinfo
    processor	: 0^M^M
    cpu		: POWER9, altivec supported^M^M
    clock		: 512.000000MHz^M^M
    revision	: 2.0 (pvr 004e 1200)^M^M
    ^M^M
    timebase	: 512000000^M^M
    platform	: PowerNV^M^M
    model		: Mambo,Simulated-System^M^M
    machine		: PowerNV Mambo,Simulated-System^M^M
    firmware	: OPAL^M^M
    MMU		: Radix^M^M
    [console-expect]#echo $?
    echo $?
    0^M^M
    [console-expect]#cat /proc/interrupts
    cat /proc/interrupts
               CPU0       ^M^M
     16:          0      XICS   2 Edge      IPI^M^M
     17:          0  OPAL EVT  11 Level     opal-msg^M^M
     18:         21  OPAL EVT   4 Edge      hvc_console^M^M
    LOC:      33666   Local timer interrupts for timer event device^M^M
    BCT:          0   Broadcast timer interrupts for timer event device^M^M
    LOC:          0   Local timer interrupts for others^M^M
    SPU:          0   Spurious interrupts^M^M
    PMI:          0   Performance monitoring interrupts^M^M
    MCE:          0   Machine check exceptions^M^M
    HMI:          0   Hypervisor Maintenance Interrupts^M^M
    NMI:          0   System Reset interrupts^M^M
    WDG:          7   Watchdog soft-NMI interrupts^M^M
    DBL:          0   Doorbell interrupts^M^M
    [console-expect]#echo $?
    echo $?
    0^M^M
    ok

    ----------------------------------------------------------------------
    Ran 1 test in 28.541s

    OK
```


*Thanks for Reading..., Hope this blog helps you in someways :-)*

<script src="https://utteranc.es/client.js"
       repo="sathnaga/sathnaga.github.io"
       issue-term="url"
       theme="github-light"
       crossorigin="anonymous"
       async>
</script>
