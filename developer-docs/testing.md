# Testing Standards in RKE2

Go testing in RKE2 comes in 2 forms: Unit, Integration. This document will 
explain *when* each test should be written and *how* each test should be
generated, formatted, and run.

Note: all shell commands given are relative to the root RKE2 repo directory.

___

## Unit Tests

Unit tests should be written when a component or function of a package needs testing.
Unit tests should be used for "white box" testing.

### Framework

All unit tests in RKE2 follow a [Table Driven Test](https://github.com/golang/go/wiki/TableDrivenTests) style.
Specifically, RKE2 unit tests are automatically generated using the [gotests](https://github.com/cweill/gotests) tool.
This is built into the Go vscode extension, has documented integrations for other popular editors, or can be run via
command line. Additionally, a set of custom templates are provided to extend the generated test's functionality.
To use these templates, call:

```bash
gotests --template_dir=<PATH_TO_RKE2>/contrib/gotests_templates
```

Or in vscode, edit the Go extension setting `Go: Generate Tests Flags`  
and add `--template_dir=<PATH_TO_RKE2>/contrib/gotests_templates` as an item.

### Format

All unit tests should be placed within the package of the file they test.  
All unit test files should be named: `<FILE_UNDER_TEST>_test.go`.
All unit test functions should be named: `Test_Unit<FUNCTION_TO_TEST>` or `Test_Unit<RECEIVER>_<METHOD_TO_TEST>`.
See the [service account unit test](https://github.com/rancher/rke2/blob/master/pkg/rke2/serviceaccount_test.go) as an example.

### Running

```bash
go test ./pkg/... -run Unit
```

Note: As unit tests call functions directly, they are the primary drivers of RKE2's code coverage metric.

___

## Integration Tests

Integration tests should be used to test a specific functionality of RKE2 that exists across multiple Go packages,
either via exported function calls, or more often, CLI comands. Integration tests should be used for "black box"
testing. 

### Framework

All integration tests in RKE2 follow a [Behavior Diven Development (BDD)](https://en.wikipedia.org/wiki/Behavior-driven_development) style.
Specifically, RKE2 uses [Ginkgo](https://onsi.github.io/ginkgo/) and [Gomega](https://onsi.github.io/gomega/) to drive the tests.  
To generate an initial test, the command `ginkgo bootstrap` can be used.

To facilitate RKE2 CLI testing, see `tests/util/cmd.go` helper functions.

### Format

All integration tests should be places in `tests` directory.
All integration test files should be named: `<TEST_NAME>_int_test.go`
All integration test functions should be named: `Test_Integration<Test_Name>`.
See the [etcd snapshot test](https://github.com/rancher/rke2/blob/master/tests/etcd_int_test.go) as an example.

### Running

Integration tests must be with an existing single-node cluster, tests will skip if the server is not configured correctly.
```bash
make dev-shell
# Once in the dev-shell
# Start rke2 server with appropriate flags
./bin/rke2 server 
```
Open another terminal
```bash
make dev-shell-enter
# once in the dev-shell
go test ./tests/ -run Integration
```
___

## Installer Tests

Installer tests should be run anytime one makes a modification to the installer script(s).
These can be run locally with Vagrant:
- [CentOS 7](../tests/vagrant/install/centos-7) (stand-in for RHEL 7)
- [CentOS 8](../tests/vagrant/install/centos-8) (stand-in for RHEL 8)
- [MicroOS](../tests/vagrant/install/opensuse-microos) (stand-in for SLE-Micro)
- [Ubuntu 20.04](../tests/vagrant/install/ubuntu-focal) (Focal Fossa)

When adding new installer test(s) please copy the prevalent style for the `Vagrantfile`.
Ideally, the boxes used for additional assertions will support the default `virtualbox` provider which
enables them to be used by our Github Actions Workflow(s). See [install.yaml](../.github/workflows/install.yaml).

### Framework

If you are new to Vagrant, Hashicorp has written some pretty decent introductory tutorials and docs, see:
- https://learn.hashicorp.com/collections/vagrant/getting-started
- https://www.vagrantup.com/docs/installation

#### Plugins and Providers

The `libvirt` and `vmware_desktop` providers cannot be used without first [installing the relevant plugins](https://www.vagrantup.com/docs/cli/plugin#plugin-install)
which are [`vagrant-libvirt`](https://github.com/vagrant-libvirt/vagrant-libvirt) and
[`vagrant-vmware-desktop`](https://www.vagrantup.com/docs/providers/vmware/installation), respectively.
Much like the default [`virtualbox` provider](https://www.vagrantup.com/docs/providers/virtualbox) these will do
nothing useful without also installing the relevant server runtimes and/or client programs.

#### Environment Variables 

These can be set on the CLI or exported before invoking Vagrant:
- `TEST_INSTALL_SH` (default :arrow_right: `../../../../install.sh`)
  The location of the installer script to be uploaded to the guest by the [Vagrant File Provisioner](https://www.vagrantup.com/docs/provisioning/file) before being invoked at `/home/vagrant/install.sh`.
- `TEST_VM_CPUS` (default :arrow_right: 2)
  The number of vCPU for the guest to use.
- `TEST_VM_MEMORY` (default :arrow_right: 2048)
  The number of megabytes of memory for the guest to use.
- `TEST_VM_BOOT_TIMEOUT` (default :arrow_right: 600)
  The time in seconds that Vagrant will wait for the machine to boot and be accessible.
- `INSTALL_PACKAGES`
  A space-delimited string of package names (or locations accessible to the guest's native package manager) that will 
  be installed prior to invoking the RKE2 install script. Useful for overriding the `rke2-selinux` RPM via URL pointing
  at a specific Github release artifact.
- `INSTALL_RKE2_*`, `RKE2_*`
  All values (other than those mentioned below) are passed through to the guest for capture by the installer script:
    - `INSTALL_RKE2_TYPE=server` (hard-coded)
    - `RKE2_KUBECONFIG_MODE=0644` (hard-coded)
    - `RKE2_TOKEN=vagrant` (hard-coded)

___

## Contributing New Or Updated Tests

___
If you wish to create a new test or update an existing test, 
please submit a PR with a title that includes the words `<NAME_OF_TEST> (Created/Updated)`.
