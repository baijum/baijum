# Sandbox

## Operators

- [Host Operator](https://github.com/codeready-toolchain/host-operator)
- [Member Operator](https://github.com/codeready-toolchain/member-operator)

## Setting up Locally using CRC for KAAS Host server

### Preparation in the Physical machine

```
sudo apt-get install -y virt-manager network-manager
```

Create a VM using Debian 12 iso image (Fedora also should work).

Inside the VM:

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

Install Apache for reverse proxy.

```
sudo apt-get install apache2
```

Enable these mods:

```
sudo a2enmod ssl rewrite proxy proxy_http proxy_ajp
```

Generate certificate:

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
```

Create VirtualHost `/etc/apache2/sites-available/kaas-host.muthukadan.net.conf`:

```
<VirtualHost *:443>
   ServerName kaas-host.muthukadan.net.conf

   ProxyPreserveHost On
   SSLEngine on
   SSLProxyEngine on
   SSLProxyVerify none
   SSLProxyCheckPeerCN off
   SSLProxyCheckPeerName off
   SSLProxyCheckPeerExpire off
   ProxyPass / https://api.crc.testing:6443/
   ProxyPassReverse / https://api.crc.testing:6443/

   SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
   SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
</VirtualHost>
```

a2ensite kaas-host.muthukadan.net.conf

## Setting up Locally using CRC for KAAS Member server

Follow the similar steps as that of Host server, but change the domain names.

