# Run any host tests on IBM OpenPower boxes using op-test-framework.

## Introduction to op-test-framework:
[op-test-framework](https://github.com/open-power/op-test-framework) is a python unittest based test suite for validating IBM OpenPower boxes, which comprises many tests including booting host with multiple configurations etc.

## Test enables the support:
[RunHostTest.py](https://github.com/open-power/op-test-framework/blob/master/testcases/RunHostTest.py)

## Why to run other framework tests through op-test-framework:
op-test-framework has it own benefits like
1. _Enable console support for IBM Power boxes_
2. _Automatic Runtime failure detection like host crash_
3. _Power cycle support_
4. _pre-written libraries for various IPMI commands_

But we might already have testcases reside in other frameworks,like [Avocado](https://sathnaga.github.io/2018/05/17/testing-kvm-on-power-using-avocado-test.html).
and shortcomings of those framework would be, environment will get lost,
when a testcase crashes the host and most of similar tests are designed to run within host.

So instead of rewriting whole test in op-test-framework to enjoy its features.
 We can just run tests from those frameworks through op-test-framework.

## How to use it:

__We can use it two ways:__
* run a single command in host using option (--host-cmd)
* run multiple commands by supplying a file as input where each line is considered as a command and it detects `reboot` as a special line and takes care of host power cycle.(--host-cmd-file)

__Sample configuration file:__
```
$cat host_tests.conf
[op-test]
bmc_type=BMC
bmc_ip=x.x.x.x
bmc_username=ADMIN
bmc_password=ADMIN
bmc_usernameipmi=ADMIN
bmc_passwordipmi=ADMIN
host_ip=x.x.x.x
host_user=root
host_password=passw0rd
host_cmd_timeout=36000
host_cmd_file=./tests.conf
machine_state=OS <
```

__Explanation to above op-test config file:__
```
host and bmc credentials which is common for most of the tests and
host_cmd_timeout - timeout for each command in the file(just given maximum timeout)
host_cmd_file - file path, relative to op-test-framework base directory
```

__An example host_cmd_file:__
* compiles upstream guest kernel, boots and runs a KVM guest vcpuhotplug test.
```
$cat tests.conf
[ -d /home/kvmci/ ] || mkdir -p /home/kvmci
echo "KVMCI: Building Upstream guest kernel..."
[ -d /home/kvmci/linux ] || (cd /home/kvmci && git clone --depth 1 https://git.kernel.org/pub/scm/linux/kernel/git/powerpc/linux.git -b merge)
cd /home/kvmci/linux
make ppc64le_guest_config
make olddefconfig
make -j 240
avocado run libvirt_vcpu_plug_unplug.positive_test.vcpu_set.guest.vm_operate.no_operation --vt-type libvirt --vt-extra-params create_vm_libvirt=yes kill_vm_libvirt=yes env_cleanup=yes mem=20480 smp=2 take_regular_screendumps=no backup_image_before_testing=no libvirt_controller=virtio-scsi scsi_hba=virtio-scsi-pci drive_format=scsi-hd use_os_variant=no restore_image_after_testing=no vga=none display=nographic kernel=/home/kvmci/linux/vmlinux kernel_args='root=/dev/sda2 rw console=tty0 console=ttyS0,115200 init=/sbin/init initcall_debug selinux=0' --vt-guest-os JeOS.27.ppc64le --job-results-dir /home/kvmci/
```

## Let's run:
```
$git clone https://github.com/open-power/op-test-framework
$cd op-test-framework
$./op-test --run testcases.RunHostTest.RunHostTest -c ./host_tests.conf

...
...

[console-expect]#avocado run libvirt_vcpu_plug_unplug.positive_test.vcpu_set.guest.vm_operate.no_operation --vt-type libvirt --vt-extra-params create_vm_libvirt=yes kill_vm_libvirt=yes env_cleanup=yes mem=20480 smp=2 take_regular_screendumps=no backup_image_before_testing=no libvirt_controller=virtio-scsi scsi_hba=virtio-scsi-pci drive_format=scsi-hd use_os_variant=no restore_image_after_testing=no vga=none display=nographic kernel=/home/kvmci/linux/vmlinux kernel_args='root=/dev/sda2 rw console=tty0 console=ttyS0,115200 init=/sbin/init initcall_debug selinux=0' --vt-guest-os JeOS.27.ppc64le --job-results-dir /home/kvmci/
<nel=/home/kvmci/linux/vmlinux kernel_args='root=/dev/sda2 rw console=tty0 console=ttyS0,115200 init=/sbin/init initcall_debug selinux=0' --vt-guest-os JeOS.27.ppc64le --job-results-dir /home/kvmci/
JOB ID     : b291eb36bb522ff6a3c4d8db1b9124368e20e2bc
JOB LOG    : /home/kvmci/job-2018-11-09T00.19-b291eb3/job.log
 (1/1) type_specific.io-github-autotest-libvirt.libvirt_vcpu_plug_unplug.positive_test.vcpu_set.guest.vm_operate.no_operation:  PASS (75.76 s)
RESULTS    : PASS 1 | ERROR 0 | FAIL 0 | SKIP 0 | WARN 0 | INTERRUPT 0 | CANCEL 0
JOB TIME   : 78.36 s
JOB HTML   : /home/kvmci/job-2018-11-09T00.19-b291eb3/results.html
[console-expect]#echo $?
echo $?
0
[console-expect]#Warning: Permanently added '9.40.192.88' (ECDSA) to the list of known hosts.
OK (113.264s)

----------------------------------------------------------------------
Ran 1 test in 113.265s

OK

Generating XML reports...
```

References:

[Guest kernel config](http://patchwork.ozlabs.org/patch/994647/)

[Avocado KVM Tests](https://sathnaga.github.io/2018/05/17/testing-kvm-through-libvirt-environment.html)

Hope this helps you :-), let's break the boxes with tests...
