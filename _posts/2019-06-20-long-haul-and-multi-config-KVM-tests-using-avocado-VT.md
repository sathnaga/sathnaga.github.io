# Long Haul and multi configuration KVM guests testing using Avocado-VT

With reducing time to market, it becomes crucial we attempt the long haul and
stress tests mimicking the real world scenarios, and it is difficult to do it manually with less time in hand.
So, here comes the proven KVM test suite Avocado-VT, where we have implemented
recently a lot of capabilities which will allow any one to run not only longevity
tests but also in parallel multiple Guest VM with different configurations.


My previous [blog](https://sathnaga86.com/) explain how to configure and use Avocado-VT KVM test suites.

Here let us see how to run some of long haul and multi configuration tests:

_**Scenario 1**: User wants to check whether guest withstand huge number of
reboot requests, lets take some really huge number, 5000._

Solution: Here comes the simple avocado command line allow us to accomplish it,
```
avocado run io-github-autotest-qemu.reboot \
--vt-type libvirt \
--vt-guest-os JeOS.27.ppc64le \
--job-results-dir /home/sath \
--vt-extra-params qemu_binary=/home/sath/qemu/ppc64-softmmu/qemu-system-ppc64 \
emulator_path=//home/sath/qemu/ppc64-softmmu/qemu-system-ppc64 \
mem=81920 smp=256 vcpu_maxcpus=256 \
take_regular_screendumps=no backup_image_before_testing=no \
libvirt_controller=virtio-scsi scsi_hba=virtio-scsi-pci drive_format=scsi-hd \
use_os_variant=no restore_image_after_testing=no \
vga=none display=nographic \
kernel=/home/sath/linux/vmlinux kernel_args='root=/dev/sda2 rw console=tty0 console=ttyS0,115200 init=/sbin/init initcall_debug selinux=0' \
virtinstall_qemu_cmdline=' -M pseries,ic-mode=xive' \
initrd='' \
use_serial_login=yes \
kill_vm=yes kill_vm_libvirt=yes env_cleanup=yes create_vm_libvirt=yes \
test_timeout=1440000 \
reboot_count=5000

JOB ID     : 73013bc0a7102829a87786bb6ba3efd35e816cc0
JOB LOG    : /home/sath/job-2019-05-18T08.17-73013bc/job.log
 (1/1) io-github-autotest-qemu.reboot: PASS (201977.56 s)
RESULTS    : PASS 1 | ERROR 0 | FAIL 0 | SKIP 0 | WARN 0 | INTERRUPT 0 | CANCEL 0
JOB TIME   : 201979.43 s
JOB HTML   : /home/sath/job-2019-05-18T08.17-73013bc/results.html
```
Explain on this test w.r.t command line parameters:

    This test reboots a KVM guest booted with given upstream kernel and qemu parameters
    in libvirt environment for user defined iterations, in this case `reboot_count=5000`
    times on given upstream qemu.

Above test ran for almost **56 hours, i.e more than 2 days**, Avocado-VT framework is
stable enough to facilitate such environment and tests.


_**Scenario 2**: User wants to check similar events say reboot
(it can be any vm events like cpu hotplug, vcpu pining, suspend/resume or all together)
in parallel for multiple guest with different configurations, like different machine types, cpu, memory setting etc._

Solution: Another simple avocado command line allow us to accomplish it,
```
avocado run multivm_cpustress.custom_host_events.custom_vm_events \
--vt-type libvirt \
--vt-guest-os JeOS.27.ppc64le \
--job-results-dir /home/sath \
--vt-extra-params qemu_binary=/home/sath/qemu/ppc64-softmmu/qemu-system-ppc64 \
emulator_path=/home/sath/qemu/ppc64-softmmu/qemu-system-ppc64 \
mem=81920 \
smp=512 \
take_regular_screendumps=no backup_image_before_testing=no \
libvirt_controller=virtio-scsi scsi_hba=virtio-scsi-pci drive_format=scsi-hd \
use_os_variant=no \
restore_image_after_testing=no \
vga=none \
display=nographic \
kernel=/home/sath/linux/vmlinux \
kernel_args='root=/dev/sda2 rw console=tty0 console=ttyS0,115200 init=/sbin/init initcall_debug selinux=0' \
virtinstall_qemu_cmdline_vm1=' -M pseries,ic-mode=dual' \
virtinstall_qemu_cmdline_vm2=' -M pseries,ic-mode=xive' \
virtinstall_qemu_cmdline_vm3=' -M pseries,ic-mode=xics' \
virtinstall_qemu_cmdline_vm4=' -M pseries,ic-mode=dual,max-cpu-compat=power8' \
virtinstall_qemu_cmdline_vm5=' -M pseries,ic-mode=dual,kernel-irqchip=off' \
virtinstall_qemu_cmdline_vm6=' -M pseries,ic-mode=xive,kernel-irqchip=off' \
virtinstall_qemu_cmdline_vm7=' -M pseries,ic-mode=xics,kernel-irqchip=off' \
virtinstall_qemu_cmdline_vm8=' -M pseries,ic-mode=dual,max-cpu-compat=power8,kernel-irqchip=off' \
initrd='' \
use_serial_login=yes \
kill_vm=yes kill_vm_libvirt=yes env_cleanup=yes create_vm_libvirt=yes \
vcpu_maxcpus=512 \
test_timeout=1440000 \
vms='vm1 vm2 vm3 vm4 vm5 vm6 vm7 vm8' \
stress_events='reboot' \
stress_itrs=10 \
vcpu_cores_vm1=128 vcpu_threads_vm1=4 \
vcpu_cores_vm2=256 vcpu_threads_vm2=2 \
vcpu_cores_vm3=64 vcpu_threads_vm3=8 \
vcpu_cores_vm4=512 vcpu_threads_vm4=1 \
vcpu_cores_vm5=128 vcpu_threads_vm5=4 \
vcpu_cores_vm6=256 vcpu_threads_vm6=2 \
vcpu_cores_vm7=64 vcpu_threads_vm7=8 \
vcpu_cores_vm8=512 vcpu_threads_vm8=1 \
guest_stress=no

JOB ID     : dcf3a14bae9d4007735a49a250114a6361dcea9f
JOB LOG    : /home/sath/job-2019-06-20T04.52-dcf3a14/job.log
 (1/1) type_specific.io-github-autotest-libvirt.multivm_cpustress.custom_host_events.custom_vm_events:  PASS (1214.78 s)
RESULTS    : PASS 1 | ERROR 0 | FAIL 0 | SKIP 0 | WARN 0 | INTERRUPT 0 | CANCEL 0
JOB TIME   : 1216.73 s
JOB HTML   : /home/sath/job-2019-06-20T04.52-dcf3a14/results.html
```
Explain on this test w.r.t command line parameters:

    This test reboots 8 KVM guests booted with given upstream kernel,each with
    different interrupt controller and different cpu topology in parallel
    for given user iterations, `stress_itrs=10` on given upstream qemu.


If you are interested in KVM Testing, there are much more possibilities you can explore
with [Avocado-VT](https://github.com/avocado-framework/avocado-vt) and reach us through [mailing list](https://www.redhat.com/mailman/listinfo/avocado-devel) for any suggestions or queries.

 Hope this blog helps you in someways :-)
