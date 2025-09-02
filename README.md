# lima-rhce-lab

# Prerequisites

Git repo has been cloned to local macOS host. All commands will be executed from repo root folder unless stated otherwise.

## Required Tools

Following tools are minimal requirements by this project. [Homebrew](https://brew.sh/)  is use to install these tools on macOS host.
- [ ] [Lima VM](https://github.com/lima-vm/lima)
- [ ] [socket_vmnet](https://github.com/lima-vm/socket_vmnet/)
- [ ] [Git](https://git-scm.com/)


# Install tools

This step is require only to be executed once, prior deploying your lab. 

Install `Homebrew`. After install is completed, follow a prompt for additional steps.

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Install `Taskfile`

```
brew install go-task/tap/go-task
```

Install tools 

```
task brew:install
```

# Provisioning of networking

This step is required only to be executed once, prior deploying your lab. 


## Install socket_vmnet

Per Lima VM networking documentation, Homebrew installation method is not recommended for `socket_vmnet`. Therefore alternative install method is used.

Install socket_vmnet binary from tar.gz archive file

```
VERSION="$(curl -fsSL https://api.github.com/repos/lima-vm/socket_vmnet/releases/latest | jq -r .tag_name)"
FILE="socket_vmnet-${VERSION:1}-$(uname -m).tar.gz"
curl -OSL "https://github.com/lima-vm/socket_vmnet/releases/download/${VERSION}/${FILE}"
sudo tar Cxzvf / "${FILE}" opt/socket_vmnet
```


## Prepare macOS DHCP service

DHCP service in macOS host must be prepared to handover predefined static IP addresses based on MAC addresses to be assigned on machine VMs vNIC interface.

Populate DHCP configuration database on macOS host. Please note that per these instructions any possible previous DHCP configuration DB will be overwritten.

``` 
sudo cp macos/etc/bootptab /etc/.
```

Start macOS DHCP server and load its configuration DB 

```
sudo /bin/launchctl load -w /System/Library/LaunchDaemons/bootps.plist
sudo /bin/launchctl kickstart -kp system/com.apple.bootpd
```

## Prepare Lima VM shared networking configuration

Create networking configuration for Lima VM

```
mkdir -p ~/.lima/_config
cp macos/home/.lima/_config/networks.yaml ~/.lima/_config/.
```


## Prepare sudoers file

Setup sudoers file to launch socket_vmnet from Lima VM

```
limactl sudoers | sudo tee /etc/sudoers.d/lima
```

## For reference

- https://github.com/lima-vm/socket_vmnet?tab=readme-ov-file#from-binary
- https://lima-vm.io/docs/config/network/vmnet/#socket_vmnet
