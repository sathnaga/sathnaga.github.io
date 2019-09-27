# DevOps: Libvirt: qemu: kernel: CI for KVM on Power using Avocado Test framework


This blog explains about how to setup Continuous integration test for qemu, libvirt and guest kernel development using my patches to Avocado Test framework

_Refer my previous blogs for introductions of avocado test framework._

 **_Below are Commits which enabled support:_**

* 1581 Added support Libvirt package build test.
* 1582 Add qemu binary entry for powerpc.
* 1584 Minor fixes for custom boot params.
* 1602 Add libvirt build test config.
* 09d46e3 Added virt-install kernel and initrd params support.


## Steps to setup CI on KVM IBM Power Host:

__Host Environment:  `IBM Power8 runs Linux/Fedora28`__

### Installing packages needed for qemu and libvirt building....
```
# yum groupinstall "Development tools"
# yum install pixman-devel libtool rpcgen  libnl3-devel libxml2-devel mingw64-portablexdr libtirpc-devel yajl-devel device-mapper-devel libpciaccess-devel rpm-build audit-libs-devel augeas xfsprogs-devel avahi-devel bash-completion cyrus-sasl-devel dbus-devel fuse-devel glusterfs-api-devel glusterfs-devel gnutls-devel libacl-devel libattr-devel libblkid-devel libcap-ng-devel libcurl-devel libpcap-devel librados2-devel librbd1-devel libssh-devel libssh2-devel libtasn1-devel libwsman-devel ncurses-devel netcf-devel numactl-devel parted-devel readline-devel sanlock-devel scrub systemtap-sdt-devel wireshark-devel jansson-devel libiscsi-devel
```

Refer my previous blogs for installing avocado framework,
lets skip that and jump to test runs.

### Building upstream qemu...

```
# wget https://raw.githubusercontent.com/sathnaga/avocado-vt/kvmci/kvmci/ppc/qemu_build.cfg -O /home/kvmci/qemu_build.cfg
# avocado run --vt-config /home/kvmci/qemu_build.cfg
JOB ID     : ebf827140b57cd2e183b29b372d9e4c81f3e2931
JOB LOG    : /home/sath/avocado-fvt-wrapper/results/job-2018-05-22T02.37-ebf8271/job.log
(1/1) build: PASS (22.61 s)
RESULTS    : PASS 1 | ERROR 0 | FAIL 0 | SKIP 0 | WARN 0 | INTERRUPT 0 | CANCEL 0
JOB TIME   : 33.63 s
JOB HTML   : /home/sath/avocado-fvt-wrapper/results/job-2018-05-22T02.37-ebf8271/results.html
```

The above test will clone build the upstream/given qemu gitrepo in qemu_build.cfg
which can be modified as per need and softlink qemu-binary in this path `/usr/share/avocado-plugins-vt/bin/qemu`.

### Building upstream libvirt...
```
# wget https://raw.githubusercontent.com/sathnaga/avocado-vt/kvmci/kvmci/ppc/libvirt_build.cfg -O /home/kvmci/libvirt_build.cfg
# avocado run --vt-config /home/kvmci/libvirt_build.cfg
JOB ID     : a198d64a434efc67751612bcac27e214267826dc
JOB LOG    : /home/sath/avocado-fvt-wrapper/results/job-2018-05-21T13.04-a198d64/job.log
(1/1) build: PASS (517.98 s)
RESULTS    : PASS 1 | ERROR 0 | FAIL 0 | SKIP 0 | WARN 0 | INTERRUPT 0 | CANCEL 0
JOB TIME   : 746.36 s
JOB HTML   : /home/sath/avocado-fvt-wrapper/results/job-2018-05-21T13.04-a198d64/results.html
```

The above test will clone build the  upstream/given libvirt gitrepo in libvirt_build.cfg
which can be modified as per need and install/replace the system libvirt binary and
start the libvirtd service.

### Building upstream Guest VM kernel...
```
# [ -d /home/linux ] || (mkdir -p /home/ && cd /home/ && git clone https://github.com/torvalds/linux.git)
# cd /home/linux
# make ppc64le_guest_defconfig
# make -j 240 -s
```

The above steps as shown will compile the upstream kernel,
the config used here is tweaked to accommodate the modules in built in vmlinux
as we do not depend on initramfs for booting the guest and this is tested working with 4.17 kernel.


`# setenforce 0`  --> as libvirt will fail to boot custom qemu path due to selinux

### Running kvm test with upstream qemu, upstream libvirt and upstream guest kernel...
```
# avocado run guestpin.with_emualorpin.sequential.positive.with_cpu_hotplug --vt-type libvirt --vt-extra-params emulator_path=/usr/share/avocado-plugins-vt/bin/qemu create_vm_libvirt=yes kill_vm_libvirt=yes env_cleanup=yes smp=2 backup_image_before_testing=no libvirt_controller=virtio-scsi scsi_hba=virtio-scsi-pci drive_format=scsi-hd use_os_variant=no restore_image_after_testing=no vga=none display=nographic kernel=/home/linux/vmlinux kernel_args='root=/dev/sda2 rw console=tty0 console=ttyS0,115200 init=/sbin/init  initcall_debug' take_regular_screendumps=no --vt-guest-os JeOS.27.ppc64le
JOB ID     : 569dd1a8acf99002ce4c3ae5d1702b6f28b40428
JOB LOG    : /home/sath/avocado-fvt-wrapper/results/job-2018-05-23T11.56-569dd1a/job.log
 (1/1) type_specific.io-github-autotest-libvirt.guestpin.with_emualorpin.sequential.positive.with_cpu_hotplug: PASS (108.52 s)
RESULTS    : PASS 1 | ERROR 0 | FAIL 0 | SKIP 0 | WARN 0 | INTERRUPT 0 | CANCEL 0
JOB TIME   : 119.89 s
JOB HTML   : /home/sath/avocado-fvt-wrapper/results/job-2018-05-23T11.56-569dd1a/results.html
```

The above test uses the artifact(binaries) that got generated as outcome of the previous steps
and runs a sample KVM test with upstream qemu, libvirt and guest kernel.

I have written a shell script which does above described steps except dependency package installation,
this can be executed on a IBM PowerKVM host and get the KVM CI working!!!

```
# wget https://raw.githubusercontent.com/sathnaga/avocado-vt/kvmci/kvmci/ppc/kvmci_hostcmd.sh
#./kvmci_hostcmd.sh
KVMCI: Building Upstream Kernel...
KVMCI: Installing avocado...
KVMCI: Installing avocado-vt...
KVMCI: Bootstrapping avocado-vt...
KVMCI: Building Upstream Qemu...
JOB ID     : 7effbff2eb661a4979c58e7891158fd183cd5f95
JOB LOG    : /root/avocado/job-results/job-2018-05-24T06.15-7effbff/job.log
 (1/1) build: PASS (22.50 s)
RESULTS    : PASS 1 | ERROR 0 | FAIL 0 | SKIP 0 | WARN 0 | INTERRUPT 0 | CANCEL 0
JOB TIME   : 32.53 s
JOB HTML   : /root/avocado/job-results/job-2018-05-24T06.15-7effbff/results.html
KVMCI: Building Upstream Libvirt...
JOB ID     : 5992e90c9a1ebf4ebd8b53758c986bd2df4fb30b
JOB LOG    : /root/avocado/job-results/job-2018-05-24T06.15-5992e90/job.log
 (1/1) build: PASS (515.30 s)
RESULTS    : PASS 1 | ERROR 0 | FAIL 0 | SKIP 0 | WARN 0 | INTERRUPT 0 | CANCEL 0
JOB TIME   : 747.75 s
JOB HTML   : /root/avocado/job-results/job-2018-05-24T06.15-5992e90/results.html
KVMCI: Running tests with Upstream qemu, libvirt, guest kernel...
JOB ID     : 3c21388a07c43c0bc6efdc18ca869c12f8ec4efd
JOB LOG    : /root/avocado/job-results/job-2018-05-24T06.28-3c21388/job.log
 (1/1) type_specific.io-github-autotest-libvirt.guestpin.with_emualorpin.sequential.positive.with_cpu_hotplug: PASS (108.77 s)
RESULTS    : PASS 1 | ERROR 0 | FAIL 0 | SKIP 0 | WARN 0 | INTERRUPT 0 | CANCEL 0
JOB TIME   : 125.67 s
JOB HTML   : /root/avocado/job-results/job-2018-05-24T06.28-3c21388/results.html
```

_Hope this blog help you in someway, Thanks for reading..._
