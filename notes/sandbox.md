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

Login to crc:

```
eval $(crc oc-env)
crc console --credentials
oc login ...
```

Generate certificate:

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
openssl genrsa -out apache-selfsigned.key 3072
openssl req -new -out apache-selfsigned.csr -sha256 -key apache-selfsigned.key -subj "/CN=kaas-host.192-168-1-35.sslip.io" -addext "subjectAltName=DNS:apps.kaas-host.192-168-1-35.sslip.io,DNS:*.apps.kaas-host.192-168-1-35.sslip.io,DNS:api.kaas-host.192-168-1-35.sslip.ioDNS:oauth-openshift.kaas-host.192-168-1-35.sslip.io"
openssl x509 -req -in apache-selfsigned.csr -days 365 -signkey apache-selfsigned.key -out apache-selfsigned.crt -outform PEM
cat apache-selfsigned.key apache-selfsigned.crt > apache-selfsigned.pem
cp apache-selfsigned.key /etc/ssl/private/apache-selfsigned.key
cp apache-selfsigned.crt /etc/ssl/private/apache-selfsigned.crt
cp apache-selfsigned.pem /etc/ssl/private/apache-selfsigned.pem
```

```
$ oc create secret tls sslip-secret --cert=apache-selfsigned.crt --key=apache-selfsigned.key -n openshift-config

$ cat <<EOF > ingress-patch.yaml
spec:
  appsDomain: kaas-host.192-168-1-35.sslip.io
  componentRoutes:
  - hostname: console-openshift-console.kaas-host.192-168-1-35.sslip.io
    name: console
    namespace: openshift-console
    servingCertKeyPairSecret:
      name: sslip-secret
  - hostname: oauth-openshift.kaas-host.192-168-1-35.sslip.io
    name: oauth-openshift
    namespace: openshift-authentication
    servingCertKeyPairSecret:
      name: sslip-secret
EOF
```

```
oc patch ingresses.config.openshift.io cluster --type=merge --patch-file=ingress-patch.yaml
```

```
oc patch apiserver cluster --type=merge -p '{"spec":{"servingCerts": {"namedCertificates":[{"names":["api.kaas-host.192-168-1-35.sslip.io"],"servingCertificate": {"name": "sslip-secret"}}]}}}'
oc patch -p '{"spec": {"host": "default-route-openshift-image-registry.kaas-host.192-168-1-35.sslip.io"}}' route default-route -n openshift-image-registry --type=merge
```

Install Apache for reverse proxy.

```
sudo apt-get install apache2
```

Enable these mods:

```
sudo a2enmod ssl rewrite proxy proxy_http proxy_ajp
```

Create VirtualHost `/etc/apache2/sites-available/kaas-host.192-168-1-35.sslip.io.conf`:

```
<VirtualHost *:443>
   ServerName kaas-host.192-168-1-35.sslip.io

   ProxyPreserveHost On
   SSLEngine on
   SSLProxyEngine on
   SSLProxyVerify none
   SSLProxyCheckPeerCN off
   SSLProxyCheckPeerName off
   SSLProxyCheckPeerExpire off
   ProxyPass / https://kaas-host.192-168-1-35.sslip.io:6443/
   ProxyPassReverse / https://kaas-host.192-168-1-35.sslip.io:6443/

   SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
   SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
   SSLProxyMachineCertificateFile  /etc/ssl/private/apache-selfsigned.pem
</VirtualHost>
```

a2ensite kaas-host.192-168-1-35.sslip.io.conf

SSH:

```
ssh -i ~/.crc/machines/crc/id_ecdsa -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null  core@192.168.130.11
```

## Setting up Locally using CRC for KAAS Member server

Follow the similar steps as that of Host server, but change the domain names.

## Domains

- https://theboroer.github.io/localtunnel-www/
- https://ngrok.com/
- http://localhost.run/
- https://nip.io/
- https://sslip.io/
- https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/install-and-setup/tunnel-guide/remote/
