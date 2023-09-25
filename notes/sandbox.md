# Sandbox

## Operators

- [Host Operator](https://github.com/codeready-toolchain/host-operator)
- [Member Operator](https://github.com/codeready-toolchain/member-operator)

## Setting up Locally using CRC

```
sudo apt-get install -y virt-manager network-manager
```

Update `/etc/apparmor.d/libvirt/TEMPLATE.qemu ` with this text:

```
#
# This profile is for the domain whose UUID matches this file.
#

#include <tunables/global>

profile LIBVIRT_TEMPLATE flags=(attach_disconnected) {
  #include <abstractions/libvirt-qemu>
  /home/baiju/.crc/cache/crc_libvirt_4.12.13_amd64/crc.qcow2 rk,
}
```

Create a KVM/QEMU based VM using virt-manager.

Install CRC using these commands:

```
wget -c https://developers.redhat.com/content-gateway/rest/mirror/pub/openshift-v4/clients/crc/latest/crc-linux-amd64.tar.xz
tar Jxvf crc-linux-amd64.tar.xz
mkdir bin
cp crc-linux-2.26.0-amd64/crc bin/
crc config set consent-telemetry no
crc setup
crc start
```

