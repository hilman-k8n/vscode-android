# vscode-android
Run vscode on android device


# ToC
- [Download Termux](#download-termux)
- [Setup storage](#setup-storage)
- [Install proot-distro](#install-proot-distro)
- [Login to Ubuntu](#login-to-ubuntu)
- [Install deps](#install-deps)
- [Install Oh My ZSH (optional)](#install-oh-my-zsh-optional)
- [Update ZSH plugins (optional)](#update-zsh-plugins-optional)
- [Install VS Code Server](#install-vs-code-server)
- [Start VS Code Server](#start-vs-code-server)
- [Install ASDF (optional)](#install-asdf-optional)
  - [Install ASDF CLI](#install-asdf-cli)
  - [Install Development tools](#install-development-tools)
  - [Install gcloud](#install-gcloud)
- [Custom Alias (optional)](#custom-alias-optional)
- [Update `~/.zshrc`](#update-zshrc)
- [References](#references)

## Download Termux
Download and install termux from [release](https://github.com/termux/termux-app/releases) page

## Setup storage
Open termux and run these command
```bash
termux-setup-storage
```

## Install proot-distro
```bash
pkg install proot-distro
proot-distro install ubuntu
```

## Login to Ubuntu
```bash
proot-distro login ubuntu
```

## Install deps
```bash
PKG_DEPS="git vim curl dnsutils jq procps unzip wget"
apt update
echo $PKG_DEPS | xargs apt install -y
```

## Install Oh My ZSH (optional)
```bash
apt update && apt install -y zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## Update ZSH plugins (optional)
```bash
ZSH_PLUGIN_LIST="git kubectl kubectx kube-ps1 aws"
sed -i "s/^plugins=.*/plugins=($ZSH_PLUGIN_LIST)/g" ~/.zshrc
```

## Install VS Code Server
```bash
wget -LO vscode.tar.gz 'https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-arm64'
tar -xvf vscode.tar.gz
mv code /usr/local/bin/code
```

## Start VS Code Server
```bash
code tunnel --accept-server-license-terms
```
You can also run the tunnel using `code-tunnel` command. See [Custom Alias (optional)](#custom-alias-optional)

## Install ASDF (optional)
### Install ASDF CLI
```bash
git clone https://github.com/asdf-vm/asdf.git ~/.asdf --branch v0.12.0
```
Update `~/.zshrc` file (see [Update ~/.zshrc](#update-zshrc))

### Install Development tools
```
apt install -y build-essential locales
locale-gen "en_US.utf8"

apt install -y python3

asdf plugin add awscli
asdf plugin add helm
asdf plugin add helmfile
asdf plugin add java
asdf plugin add k9s
asdf plugin add krew
asdf plugin add kubectl
asdf plugin add kubectx
asdf plugin add saml2aws
asdf plugin add terraform
asdf plugin add yq

cat << EOF > $HOME/.tool-versions
awscli 2.13.12
helm 3.12.3
helmfile 0.149.0
kubectl 1.24.0
kubectx 0.9.4
k9s 0.26.7
saml2aws 2.36.2
krew 0.4.3
java temurin-18.0.0+36
yq 4.30.8
terraform 1.5.5
EOF

asdf install
```

### Install gcloud
```bash
cd /opt
curl -o /opt/gcloud.tar.gz https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-443.0.0-linux-arm.tar.gz
tar -xvf gcloud.tar.gz
./google-cloud-sdk/install.sh

# GKE auth plugin for GKE 1.26+
gcloud components install gke-gcloud-auth-plugin
```

## Custom Alias (optional)
Create alias
```bash
cat << 'EOF' > $HOME/.custom-alias
#!/bin/sh

# VS code server
alias code-tunnel='code tunnel --accept-server-license-terms'

# kill SSH tunnel
tunnel-kill() {
    ps -ef | grep 'bin/ssh' | awk '{print $2}' | xargs kill -9
}

# start SSH tunnel with Google Cloud IAP
tunnel-start() {
    instance_prefix=${1:=gke}
    tunnel_port=${2:=8080}
    local instance=$(gcloud compute instances list | grep ${instance_prefix} | head -n1 | awk '{print $1}')
    gcloud compute ssh --tunnel-through-iap  --ssh-flag="-D :${tunnel_port} -fN -o TCPKeepAlive=yes -o ServerAliveInterval=5" $instance
}
EOF
```
## Update `~/.zshrc`
Update `~/.zshrc` (needs [oh my zsh](#install-oh-my-zsh-optional))
```
sed -i 's/^# User configuration/# User configuration\nsource $HOME\/.custom-alias\n. $HOME\/.asdf\/asdf.sh\nsource $HOME\/.krew\/bin\nPROMPT='$(kube_ps1)'$PROMPT/g' ~/.zshrc
```

## Troubleshooting
### Fix `character not in range` issue
If during proot startup there is an output like `/root/.oh-my-zsh/plugins/kube-ps1/kube-ps1.plugin.zsh:27: character not in range`, Execute the following command:
```
sed -i 's/^# Path to your oh-my-zsh installation./# Path to your oh-my-zsh installation.\nexport LC_ALL=en_US.UTF-8\nexport LANG=en_US.UTF-8/g' ~/.zshrc
 
```

## References
- The VSCode Server Blog Post [https://code.visualstudio.com/blogs/2022/07/07/vscode-server](https://code.visualstudio.com/blogs/2022/07/07/vscode-server)
