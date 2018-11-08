# Install/boot Upstream Kernel on IBM OpenPower boxes using op-test-framework.


## Introduction to op-test-framework:
[op-test-framework](https://github.com/open-power/op-test-framework) is a python unittest based test suite for validating IBM OpenPower boxes, which comprises many tests including booting host with multiple configurations etc.

## My Patch to enable the test:
[https://github.com/open-power/op-test-framework/pull/293 ](https://github.com/open-power/op-test-framework/pull/293) \*yet to be merged

## What does this patch do:
This test takes input of upstream(any) kernel git repository, branch and optionally a patch to be applied on top of the given git repository and it builds, installs and boot that kernel into given IBM OpenPower box and makes it ready for further tests on it.


## How to use it:
Lets see how to build, install and boot upstream kernel on IBM Power Host.

1. Clone op-test framework repository in any of the remote system(not the SUT)

```
$git clone https://github.com/open-power/op-test-framework

$cd op-test-framework

* for now the above mentioned patch to be applied
$wget https://patch-diff.githubusercontent.com/raw/open-power/op-test-framework/pull/293.patch;git am 293.patch
```

2. Create a machine.conf to match the host(SUT) that you want to install upstream kernel like below

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

```
Explanation of config file
- git_repo - upstream kernel repository to be used
- git_branch - brach of upstream kernel repository to be used
- git_repoconfigpath(optional) - kernel config can be supplied as url or local path, by default `ppc64le_defconfig` will be used
- git_patch(optional) - any incremental patch to be applied on top of above kernel repository
- git_home(optional) - base path for kernel repository clone, by default `/home/ci` will be used
```

3. Running Upstream Kernel Install test....

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

Now your server has the upstream kernel installed and ready to test :-) ...
Hope this helps, Thanks for reading...
