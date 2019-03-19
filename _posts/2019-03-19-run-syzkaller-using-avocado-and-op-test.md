# This blog explains how avocado and op-test helps us to setup and run syzkaller on Power Platform

## **What is syzkaller:**
[syzkaller](https://github.com/google/syzkaller) is an unsupervised, coverage-guided kernel fuzzer

## **Avocado test enables support:**

Testcase: [syzkaller.power](https://github.com/autotest/tp-qemu/pull/1691) *

    *  Pull request yet to be merged, till it gets merged this patch needs to applied manually before running below avocado command line.

### *What the testcase do:*

1. Install/Setup syzkaller in host.
2. Setup Guest passwordless ssh from host.
3. Prepare and compile Guest kernel.
4. Prepare syzkaller config with user given qemu params and guest params.
5. Start sykaller with above config and run for specified time(test_timeout).
6. Fails out the test in case of host issues.

### *syzkaller specific testcase config parameters:*

Below syzkaller specific test parameters with default values as given here,
which can be modified in avocado command line to match as per our needs using
`--vt-extra-params` as other params as mentioned in below avocado command line.
More details on the syzkaller config params can be found under [here](https://github.com/google/syzkaller/blob/master/docs/configuration.md)

```
syz_qemu_args = "-enable-kvm -M pseries -net nic,model=virtio"
syz_kernel_repo = "https://github.com/linuxppc/linux.git"
syz_kernel_branch = "merge"
syz_kernel_config = "ppc64le_guest_defconfig"
syz_target = "linux/ppc64le"
syz_cmd_params = "-debug -v 10"
syz_http = "0.0.0.0:56741"
syz_count = 1
syz_procs = 4
```

## **How to use:**

**_ENV Used for this blog:_**

* Host : IBM Power8 runs Fedora 28 with kernel 5.0.0
* Guest: Fedora 27 with kernel 5.1.0-rc1-g3770244b0
* Qemu: 3.1.50 (v3.1.0-3117-ga9d1cc9f56-dirty)

Below is the sample avocado command line that runs above mentioned syzkaller test

```
# avocado run syzkaller --vt-type libvirt --vt-extra-params create_vm_libvirt=yes kill_vm_libvirt=yes env_cleanup=yes mem=20480 smp=2 take_regular_screendumps=no test_timeout=4000 backup_image_before_testing=no libvirt_controller=virtio-scsi scsi_hba=virtio-scsi-pci drive_format=scsi-hd use_os_variant=no restore_image_after_testing=no vga=none display=nographic --vt-guest-os JeOS.27.ppc64le
JOB ID     : 78a8a2399ad98bd0e8999f365eff1539009e9351
JOB LOG    : /home/sath/avocado-fvt-wrapper/results/job-2019-03-18T11.58-78a8a23/job.log
 (1/1) powerkvm-qemu.syzkaller.power: PASS (1830.90 s)
RESULTS    : PASS 1 | ERROR 0 | FAIL 0 | SKIP 0 | WARN 0 | INTERRUPT 0 | CANCEL 0
JOB TIME   : 1837.14 s
```

### **_Running syzkaller testcase through op-test:_**

There are lot of chances we might hit host crash itself, so let's run above Avocado test
using [op-test-framework](https://sathnaga86.com/2018/11/11/run-host-tests-using-op-test-framework.html),
so that we can capture host console and debug logs during host crash situations.

**_op-test configuration file:_**

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
host_cmd_timeout=5000
host_cmd_file=./tests.conf
host_cmd_resultpath=/home/syzkaller/latest
machine_state=OS
```

**_test configuration file:_**

This expects avocado-vt,avocado is installed already in the system under test, if not please refer this blog to get it installed, [How to install avocado-vt](https://sathnaga86.com/2018/05/17/testing-kvm-through-libvirt-environment.html)

```
$cat tests.conf
avocado run syzkaller --vt-type libvirt --vt-extra-params create_vm_libvirt=yes kill_vm_libvirt=yes env_cleanup=yes mem=20480 smp=2 take_regular_screendumps=no backup_image_before_testing=no libvirt_controller=virtio-scsi scsi_hba=virtio-scsi-pci drive_format=scsi-hd use_os_variant=no test_timeout=4000 restore_image_after_testing=no vga=none display=nographic --vt-guest-os JeOS.27.ppc64le --job-results-dir /home/syzkaller --job-timeout 4100
```

Let's run now syzkaller test using op-test framework with above mentioned configuration files.

```
./op-test -c host_tests.conf --run testcases.RunHostTest.RunHostTest
...
...
...
[console-expect]#avocado run syzkaller --vt-type libvirt --vt-extra-params create_vm_libvirt=yes kill_vm_libvirt=yes env_cleanup=yes mem=20480 smp=2 take_regular_screendumps=no backup_image_before_testing=no libvirt_controller=virtio-scsi scsi_hba=virtio-scsi-pci drive_format=scsi-hd use_os_variant=no test_timeout=4000 restore_image_after_testing=no vga=none display=nographic --vt-guest-os JeOS.27.ppc64le --job-results-dir /home/syzkaller --job-timeout 4100
avocado run syzkaller --vt-type libvirt --vt-extra-params create_vm_libvirt=yes kill_vm_libvirt=yes env_cleanup=yes mem=20480 smp=2 take_regular_screendumps=no backup_image_befno test_timeout=4000 restore_image_after_testing=no vga=none display=nographic --vt-guest-os JeOS.27.ppc64le --job-results-dir /home/syzkaller --job-timeout 4100
JOB ID     : b9c81856ceddded5b6a90832c35afe5400091a17
JOB LOG    : /home/syzkaller/job-2019-03-19T03.10-b9c8185/job.log
 (1/1) powerkvm-qemu.syzkaller.power: PASS (3998.13 s)
RESULTS    : PASS 1 | ERROR 0 | FAIL 0 | SKIP 0 | WARN 0 | INTERRUPT 0 | CANCEL 0
JOB TIME   : 4004.33 s
JOB HTML   : /home/syzkaller/job-2019-03-19T03.10-b9c8185/results.html
[console-expect]#echo $?
echo $?
0
[console-expect]#OK (4030.016s)

----------------------------------------------------------------------
Ran 1 test in 4030.016s

OK
```

_Hope this blog help you in someway, Thanks for reading..._
