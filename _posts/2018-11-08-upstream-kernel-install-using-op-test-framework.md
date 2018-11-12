# Install/boot Upstream Kernel on IBM OpenPower boxes using op-test-framework.

## Introduction to op-test-framework:
[op-test-framework](https://github.com/open-power/op-test-framework) is a python unit-test based test suite for validating IBM OpenPower boxes, which comprises many tests including booting host with multiple configurations etc.

## Test enables the support:
[InstallUpstreamKernel.py](https://github.com/open-power/op-test-framework/blob/master/testcases/InstallUpstreamKernel.py)

## What does this test do:
This test takes input of upstream(any) kernel git repository, branch and optionally a patch to be applied on top of the given git repository and it builds, installs and boots that kernel into given IBM OpenPower box and makes it ready for further tests on it.

## How to use it:
Lets see how to build, install and boot upstream kernel on IBM Power Host.

----

__Clone op-test framework repository in any of the remote system(not the SUT):__

```
$git clone https://github.com/open-power/op-test-framework
$cd op-test-framework
```

----

__Create a machine.conf to match the host(SUT) that you want to install upstream kernel like below__

```
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
git_repo=https://github.com/linuxppc/linux.git
git_branch=merge
git_repoconfigpath=https://raw.githubusercontent.com/sathnaga/avocado-vt/kvmci/kvmci/ppc/config_kvmppc
git_patch=958675.mbox
git_home=/home/kvmci
machine_state=OS
```

>Explanation of config file:

* git_repo - Upstream kernel repository to be used.
* git_branch(optional) - Branch of upstream kernel repository to be used, by default `master`.
* git_repoconfigpath(optional) - Kernel configuration can be supplied as url or local file path, by default `ppc64le_defconfig` will be used.
* git_patch(optional) - Any incremental patch to be applied on top of above kernel repository, which can be supplied as
url or local file path.
* git_home(optional) - Base path for kernel repository clone in SUT, by default `/home/ci` will be used.

----

__Running Upstream Kernel Install test...__

```
$./op-test -c machine.conf --run testcases.InstallUpstreamKernel.InstallUpstreamKernel


...

...

[console-expect]#2018-11-08 17:57:10,997:op-test.testcases.InstallUpstreamKernel:runTest:INFO:Installed upstream kernel version: 4.20.0-rc1-g16a0961d6

...

[console-expect]#echo $?
echo $?
0
[console-expect]#OK (836.924s)
----------------------------------------------------------------------
Ran 1 test in 836.924s

OK
```

----

Now your server has the upstream kernel installed and ready to test :-) ...

Hope this helps, Thanks for reading...
