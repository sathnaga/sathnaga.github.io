# Per-patch KVM CI using open source recipes

### __Used recipes:__

[snowpatch](https://github.com/ruscur/snowpatch) - Used to listen on upstream patches from given mailing list and trigger the jenkins job.

[Jenkins](https://jenkins.io/doc/book/pipeline/) - Used to orchestrate the CI workflow using pipeline.

[op-test-framework](https://github.com/open-power/op-test-framework) - Framework used to install upstream kernel and run tests.

[Avocado framework](https://github.com/avocado-framework) - Framework used to run OS, KVM tests.

Hardware used: IBM Power8 box

### __WorkFlow:__

1. snowpatch is configured to listen on a [powerpc kernel](https://patchwork.ozlabs.org/project/linuxppc-dev/list/) mailing list and it does
create a local branch with the new patch series on the mailing list and triggers
the given jenkins job with the repository and branch details.

    `sample snowpatch output screenshot`
    ![](https://github.com/sathnaga/sathnaga.github.io/raw/master/resources/snowpatchlistenonpatch.png)

2. Jenkins pipeline as shown is configured to run below workflow

    * _Clone op-test-framework_

    * [_Build and install upstream kernel on OpenPOWER box_](https://sathnaga86.com/2018/11/08/upstream-kernel-install-using-op-test-framework.html) - built with `ppc64le_defconfig`

    * _Build upstream guest kernel_ - built with `ppc64le_guest_defconfig`

    * [_Install Avocado test framework_](https://sathnaga86.com/2018/05/17/testing-kvm-through-libvirt-environment.html)

    * _Build upstream qemu_

    * _Build upstream libvirt_

    * [_Run KVM Tests with Upstream Host Kernel, guest kernel, qemu and libvirt_](https://sathnaga86.com/2018/11/11/run-host-tests-using-op-test-framework.html)

    ![](https://github.com/sathnaga/sathnaga.github.io/raw/master/resources/kvmcipipeline.png)

3. Notify Slack on completion

    ![](https://github.com/sathnaga/sathnaga.github.io/raw/master/resources/kvmcislacknotification.png)

#### Results can be viewed in Jenkins

![](https://github.com/sathnaga/sathnaga.github.io/raw/master/resources/kvmcipipelineresult.png)

![](https://github.com/sathnaga/sathnaga.github.io/raw/master/resources/kvmcipipelinekvmtestsresult.png)


_Thanks for reading, Hope this helps you in someways..._
