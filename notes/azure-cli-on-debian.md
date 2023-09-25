# Azure CLI on Debian

```
$ cat /etc/debian_version 
bookworm/sid

$ cat /etc/apt/preferences.d/azure-cli.pref 
Package: *
Pin: Release o=azure-cli bullseye
Pin-Priority: 600

$ sudo apt-get update
$ sudo apt-get install ca-certificates curl apt-transport-https lsb-release gnupg

$ curl -sL https://packages.microsoft.com/keys/microsoft.asc |
    gpg --dearmor |
    sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null

$ echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ bullseye main" |
    sudo tee /etc/apt/sources.list.d/azure-cli.list

$ sudo apt-get update
$ sudo apt-get install azure-cli
```

I had to use `bullseye` eventhough my system has `bookworm`.
