# vscode-android
Run vscode on android device

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
PKG_DEPS="git vim curl wget"
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
ZSH_PLUGIN_LIST="git kubectl kube-ps1 aws"
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

asdf plugin add helmfile
asdf plugin add java
asdf plugin add k9s
asdf plugin add krew
asdf plugin add kubectl
asdf plugin add kubectx
asdf plugin add saml2aws
asdf plugin add yq

cat << EOF > $HOME/.tool-versions
helmfile 0.149.0
kubectl 1.24.0
kubectx 0.9.4
k9s 0.26.7
saml2aws 2.36.2
krew 0.4.3
java temurin-18.0.0+36
yq 4.30.8
EOF

asdf install
```

### Install gcloud
```bash
cd /opt
curl -o /opt/gcloud.tar.gz https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-443.0.0-linux-arm.tar.gz
tar -xvf gcloud.tar.gz
./google-cloud-sdk/install.sh
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
    local instance=$(gcloud compute instances list | grep ${instance_prefix} | head -n1 | awk '{print $1}')
    gcloud compute ssh --tunnel-through-iap  --ssh-flag='-D :8080 -fN -o TCPKeepAlive=yes -o ServerAliveInterval=5' $instance
}
EOF
```
## Update `~/.zshrc`
Update `~/.zshrc` (needs [oh my zsh](#install-oh-my-zsh-optional))
```
sed -i 's/^# User configuration/# User configuration\nsource $HOME\/.custom-alias\n. $HOME\/.asdf\/asdf.sh/g' ~/.zshrc
```

## References
- The VSCode Server Blog Post [https://code.visualstudio.com/blogs/2022/07/07/vscode-server](https://code.visualstudio.com/blogs/2022/07/07/vscode-server)
