## hwe

```
apt install --install-recommends linux-generic-hwe-22.04 -y
```

## enable bbr on ubuntu

```
modprobe tcp_bbr
echo "tcp_bbr" >> /etc/modules-load.d/modules.conf
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p

sysctl net.ipv4.tcp_available_congestion_control
sysctl net.ipv4.tcp_congestion_control
lsmod | grep bbr

```

## nginx mainline

```
sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring

curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
    | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null

gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/mainline/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list

echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
    | sudo tee /etc/apt/preferences.d/99nginx

sudo apt update
sudo apt install nginx
```

## zsh

```
apt install zsh

sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting
```

## list kernels

```
dpkg --list | egrep -i --color 'linux-image|linux-headers|linux-modules' | awk '{ print $2 }'
```

## warp-cli

```
curl -fsSL https://pkg.cloudflareclient.com/pubkey.gpg | sudo gpg --yes --dearmor --output /usr/share/keyrings/cloudflare-warp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflare-client.list

sudo apt-get update && sudo apt-get install cloudflare-warp
// https://teamname.cloudflareaccess.com/warp
```

## remove old kernels
```
#!/bin/bash
# Run this script without any param for a dry run
# Run the script with root and with exec param for removing old kernels after checking
# the list printed in the dry run

uname -a
IN_USE=$(uname -a | awk '{ print $3 }')
echo "Your in use kernel is $IN_USE"

OLD_KERNELS=$(
    dpkg --list |
        grep -v "$IN_USE" |
        grep -Ei 'linux-image|linux-headers|linux-modules' |
        awk '{ print $2 }'
)
echo "Old Kernels to be removed:"
echo "$OLD_KERNELS"

if [ "$1" == "exec" ]; then
    for PACKAGE in $OLD_KERNELS; do
        yes | apt purge "$PACKAGE"
    done
fi
```

## speedtest cli
```
sudo apt-get install curl
curl -s https://packagecloud.io/install/repositories/ookla/speedtest-cli/script.deb.sh | sudo bash
sudo apt-get install speedtest
```

## docker
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo docker run hello-world
```

## swap
```
sudo swapoff -a
sudo rm /swapfile

sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo swapon --show

echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

sysctl vm.swappiness=10
```

## nexttrace
```
curl nxtrace.org/nt |bash
```

## vps融合怪
```
curl -L https://gitlab.com/spiritysdx/za/-/raw/main/ecs.sh -o ecs.sh && chmod +x ecs.sh && bash ecs.sh
```

## acme.sh
```
# install
curl https://get.acme.sh | sh -s email=my@example.com

# issue
# 在 .acme.sh/account.conf里面，
# SAVED_CF_Key="***"
# SAVED_CF_Email=my@example.com

./.acme.sh/acme.sh --issue -d "example.com" -d "*.example.com" --dns dns_cf --keylength ec-256 --ecc

# install cert
./.acme.sh/acme.sh --installcert -d "example.com" --ecc --fullchain-file /where/to/save/crt --key-file /where/to/save/key --reloadcmd "what to do after install and renew"

# renew cert
./.acme.sh/acme.sh --renew --dns dns_cf -d "example.com" --ecc -f

```

## optimizations
```
vm.swappiness = 10
kernel.sysrq = 1

# 文件系统参数
fs.file-max = 1024000

# 网络核心参数
net.core.rmem_max = 134217728
net.core.wmem_max = 134217728
net.core.netdev_max_backlog = 250000
net.core.somaxconn = 1024000

# IPv4 参数
net.ipv4.conf.all.rp_filter = 0
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.lo.arp_announce = 2
net.ipv4.conf.all.arp_announce = 2
net.ipv4.conf.default.arp_announce = 2
net.ipv4.ip_forward = 1
net.ipv4.ip_local_port_range = 1024 65535
net.ipv4.neigh.default.gc_stale_time = 120
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_low_latency = 1
net.ipv4.tcp_fin_timeout = 10
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_keepalive_time = 10
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_sack = 1
net.ipv4.tcp_fack = 1
net.ipv4.tcp_syn_retries = 3
net.ipv4.tcp_synack_retries = 3
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.tcp_max_tw_buckets = 8192
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_mtu_probing = 1
net.ipv4.tcp_rmem = 8192 262144 536870912
net.ipv4.tcp_wmem = 4096 16384 536870912
net.ipv4.tcp_adv_win_scale = -2
#net.ipv4.tcp_collapse_max_bytes = 6291456
net.ipv4.tcp_notsent_lowat = 131072
net.ipv4.udp_rmem_min = 16384
net.ipv4.udp_wmem_min = 16384

net.ipv4.tcp_slow_start_after_idle = 0

# IPv6 参数
net.ipv6.conf.all.forwarding = 1
net.ipv6.conf.default.forwarding = 1

# 连接跟踪参数
net.nf_conntrack_max = 25000000
net.netfilter.nf_conntrack_max = 25000000
net.netfilter.nf_conntrack_tcp_timeout_time_wait = 30
net.netfilter.nf_conntrack_tcp_timeout_established = 180
net.netfilter.nf_conntrack_tcp_timeout_close_wait = 30
net.netfilter.nf_conntrack_tcp_timeout_fin_wait = 30
```

# sign old commits
```
git rebase --exec 'GIT_COMMITTER_DATE="$(git log -1 --format=%at)" git commit --amend --no-edit -n -S' -i HEAD~n
```

# switch to chinese
```
sudo apt update
sudo apt install language-pack-zh-hans language-pack-gnome-zh-hans -y
sudo update-locale LANG=zh_CN.UTF-8
```

# watchtower
```
docker run --detach \
    --name watchtower \
    --volume /var/run/docker.sock:/var/run/docker.sock \
    --restart always \
    containrrr/watchtower --run-once
```
