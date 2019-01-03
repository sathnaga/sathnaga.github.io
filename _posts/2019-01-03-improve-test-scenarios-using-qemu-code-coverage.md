_Blog on how to measuring qemu Code Coverage using avocado-vt framework and use it to improve test scenarios_

## What is Code Coverage measurement:
Code coverage is measurement of how many lines of source code blocks/branches
are executed during automated test runs and it can be done by instrumenting
the source prior using certain tools like [gcov](https://en.wikipedia.org/wiki/Gcov)
and coverage report can be generated using tools like [gcovr](https://gcovr.com/)

## How to measure qemu code coverage and view reports for test improvements:

### Pre-requisite:
Linux box installed with Avocado framework, you can do referring my previous blogs on how to
[Install avocado](https://sathnaga86.com/2018/05/17/testing-kvm-on-power-using-avocado-test.html) and [Running libvirts test using avocado](https://sathnaga86.com/2018/05/17/testing-kvm-through-libvirt-environment.html)

* Environment Used:
>Hardware: IBM Power8 Box, Operating system: Fedora29 distribution with
latest host kernel(4.20) on it.

### How to prepare qemu source for Code coverage measurement

* #### _Let's run below qemu build test to compile qemu binary with code coverage instrumentation(gcov) enabled_

    ```
    # wget https://raw.githubusercontent.com/sathnaga/avocado-vt/kvmci/kvmci/ppc/qemu_build.cfg -O qemu_build.cfg
    # avocado run --vt-config qemu_build.cfg --vt-extra-params git_repo_qemu_configure_options="--target-list=ppc64-softmmu --enable-debug --enable-gcov"
    ```

>Now we have qemu source code/binary instrumented with gcov, lets run a kvm test and measure code covered by the test.

>We need the following patch to be merged in avocado-vt to avail the support,
[Avocado-vt patch to enable qemu code coverage support](https://patch-diff.githubusercontent.com/raw/avocado-framework/avocado-vt/pull/1873.patch)

>For now you can use my [kvmci](https://github.com/sathnaga/avocado-vt/tree/kvmci) branch for avocado-vt Or directly use above patch in your avocado-vt repository using `git am <downloaded patch>`

### How to run kvm tests to get code coverage:

* #### _Let's run below vcpu hotplug kvm test to measure code coverage_

    ```
    # avocado run libvirt_vcpu_plug_unplug.positive_test.vcpu_set.guest.vm_operate.no_operation --vt-type libvirt --vt-extra-params qemu_binary=/usr/share/avocado-plugins-vt/build/qemu/ppc64-softmmu/qemu-system-ppc64 emulator_path=/usr/share/avocado-plugins-vt/build/qemu/ppc64-softmmu/qemu-system-ppc64 create_vm_libvirt=yes kill_vm_libvirt=yes env_cleanup=yes mem=20480 smp=2 take_regular_screendumps=no backup_image_before_testing=no libvirt_controller=virtio-scsi scsi_hba=virtio-scsi-pci drive_format=scsi-hd use_os_variant=no restore_image_after_testing=no vga=none display=nographic kernel=/home/kvmci/linux/vmlinux kernel_args='root=/dev/sda2 rw console=tty0 console=ttyS0,115200 init=/sbin/init initcall_debug selinux=0' gcov_qemu=yes gcov_qemu_reset=yes  gcov_qemu_compress=yes gcov_qemu_collect_cmd_opts="--exclude-directories=capstone --html --html-details" --vt-guest-os JeOS.27.ppc64le --job-results-dir /home/kvmci/
    JOB ID     : f196562eb13d65f892b4dd8c941f32d526496dc5
    JOB LOG    : /home/kvmci/job-2018-12-18T07.24-f196562/job.log
    (1/1) type_specific.io-github-autotest-libvirt.libvirt_vcpu_plug_unplug.positive_test.vcpu_set.guest.vm_operate.no_operation: PASS (169.21 s)
    RESULTS    : PASS 1 | ERROR 0 | FAIL 0 | SKIP 0 | WARN 0 | INTERRUPT 0 | CANCEL 0
    JOB TIME   : 173.94 s
    JOB HTML   : /home/kvmci/job-2018-12-18T07.24-f196562/results.html

    # ls -l /home/kvmci/job-2018-12-18T07.24-f196562/test-results/1-type_specific.io-github-autotest-libvirt.libvirt_vcpu_plug_unplug.positive_test.vcpu_set.guest.vm_operate.no_operation/
    total 11236
    drwxr-xr-x 2 root root     4096 Dec 18 07:24 data
    -rw-r--r-- 1 root root    38423 Dec 18 07:27 debug.log
    -rw-r--r-- 1 root root 11067644 Dec 18 07:27 gcov_qemu.tar.gz
    -rw-r--r-- 1 root root    18544 Dec 18 07:27 ip-sniffer.log
    ...
    ...

    2018-12-18 07:25:46,307 process          L0596 INFO | Running 'rpm -q gcovr'
    2018-12-18 07:25:46,325 process          L0428 DEBUG| [stdout] gcovr-4.1-2.fc28.noarch
    2018-12-18 07:25:46,326 process          L0684 INFO | Command 'rpm -q gcovr' finished with 0 after 0.0127010345459s
    2018-12-18 07:25:46,343 process          L0596 INFO | Running 'gcovr -j 10 -o /home/kvmci/job-2018-12-18T07.24-f196562/test-results/1-type_specific.io-github-autotest-libvirt.libvirt_vcpu_plug_unplug.positive_test.vcpu_set.guest.vm_operate.no_operation/gcov_qemu/gcov.html -s --exclude-directories=capstone --html --html-details .'
    2018-12-18 07:27:08,552 process          L0428 DEBUG| [stderr] (WARNING) GCOV produced the following errors processing /usr/share/avocado-plugins-vt/build/qemu/migration/colo-comm.gcno:
    2018-12-18 07:27:08,552 process          L0428 DEBUG| [stdout] lines: 12.2% (35112 out of 288554)
    ```

Above `gcov_qemu.tar.gz` contains detailed code coverage reports and can be viewed in browser, sample report looks like below

![](https://github.com/sathnaga/sathnaga.github.io/raw/master/resources/gcov_report.png)

## Conclusion:

Now we should be able to check what part of code gets exercised while running our individual [blackbox](https://en.wikipedia.org/wiki/Black-box_testing) [avocado tests](https://sathnaga86.com/2018/05/17/testing-kvm-on-power-using-avocado-test.html), using the same we can improve test scenarios to address missing part of code.

_Thanks for reading, hope this helps you in someways...._