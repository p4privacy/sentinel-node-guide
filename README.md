# <img src="https://user-images.githubusercontent.com/114076168/191721379-88f4b6ca-6463-4458-aab4-73d29d1bc7a0.jpg" width="35" height="35"> Sentinel Guide: How to Install a Node [![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Fp4privacy%2Fsentinel_node_guide&count_bg=%230000ff&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)

How to install a Sentinel node for newbies.
You can install a Sentinel node either at home (a Raspberry Pi 4 is fine) or on a VPS.
You need to have Linux operating system.

## Menu

* [Preliminary Operations](#preliminary-operations)
    - [Generate SSH Keygen](#generate-ssh-keygen)
    - [Update the list of available software packages](#update-the-list-of-available-software-packages)
    - [Setting up Firewall](#setting-up-firewall)
* [Install Docker](#install-docker)
* [Preparing the Docker Image](#preparing-the-docker-image)
* [Node Configuration](#node-configuration)
* [Node Management](#node-management)

## Preliminary Operations

### Generate SSH Keygen

On your computer open a terminal and generate a new ssh keygen if you do not already have one
```bash
ssh-keygen
```

Then navigate into the ssh folder and copy the content of the generated public key
```bash
cd .ssh/
cat id_ed25519.pub
```

Then go on your server or VPS and add the SSH key to it

### Update the list of available software packages
```bash
sudo apt update && sudo apt upgrade -y
```

### Setting up Firewall

Install Firewall
```bash
sudo apt-get install ufw
```

Allow Port 22
```bash
sudo ufw allow 22
```

Enable Firewall
```sh
sudo ufw enable
```

## Install Docker

Install cURL package
```bash
sudo apt-get install --yes curl
```

Get the official Docker installation script
```bash
curl -fsSL get.docker.com -o ${HOME}/get-docker.sh
```

Install Docker
```bash
sudo sh ${HOME}/get-docker.sh
```

Enable Docker
```bash
sudo systemctl enable --now docker
```

Add user to Docker group
```bash
sudo usermod -aG docker $(whoami)
```

## Preparing the Docker Image

Install Git package
```bash
sudo apt-get install --yes git
```

Clone the GitHub repository
```bash
git clone https://github.com/sentinel-official/dvpn-node.git \
    ${HOME}/dvpn-node/
```

Checkout to the latest tag
```bash
cd ~/dvpn-node && \
git fetch && \
git checkout v0.3.2
```

Build the image
```bash
docker build --file Dockerfile \
    --tag sentinel-dvpn-node \
    --force-rm \
    --no-cache \
    --compress .
```

Create a self-signed TLS certificate
```bash
sudo apt-get install --yes openssl
```

Create a certificate
```bash
openssl req -new \
    -newkey ec \
    -pkeyopt ec_paramgen_curve:prime256v1 \
    -x509 \
    -sha256 \
    -days 365 \
    -nodes \
    -out ${HOME}/tls.crt \
    -keyout ${HOME}/tls.key
```
## Node Configuration

During the procedure, when you are asked to fill some fields as country or e-mail, you can leave them blank.

Initialize the application configuration
```bash
docker run --rm \
    --volume ${HOME}/.sentinelnode:/root/.sentinelnode \
    sentinel-dvpn-node process config init
```

Open the configuration file config.toml
```bash
nano /root/.sentinelnode/config.toml
```

Edit the required fields (see the pink comments)
```diff
[chain]
# Gas adjustment factor
gas_adjustment = 1.05

# Gas limit to set per transaction
gas = 200000

# Gas prices to determine the transaction fee
gas_prices = "0.1udvpn"

# The network chain ID
id = "sentinelhub-2"

# Tendermint RPC interface for the chain
rpc_address = "https://rpc.sentinel.co:443"
@@ this is default RPC di default, find below alternative ones @@
@@ https://rpc.mathnodes.com:443 @@
@@ https://rpc.sentinel1.badgerbite.xyz:443 @@
@@ https://rpc-sentinel.itastakers.com:443 @@
@@ https://rpc.sentinel.smartnodes.one:443 @@

# Calculate the transaction fee by simulating it
simulate_and_execute = true

[handshake]
# Enable Handshake DNS resolver
enable = true

# Number of peers
peers = 8

[keyring]
# Underlying storage mechanism for keys
backend = "test"
@@ write the word "test" which will allow you to run the node with autorestart function. Alternatively use the word "file" @@

# Name of the key with which to sign
from = "key_name"
@@ replace "key_name" with your desired name and remember, this value cannot be empty! @@

[node]
# Time interval between each set_sessions operation
interval_set_sessions = "2m0s"

# Time interval between each update_sessions transaction
interval_update_sessions = "1h55m0s"

# Time interval between each set_status transaction
interval_update_status = "55m0s"

# API listen-address
listen_on = "0.0.0.0:12345"
@@ this is the TCP port, you can change it and write it down as you need it later @@

# Name of the node
moniker = "your_node_name"
@@ replace "your_node_name" with your desired name @@

# Per Gigabyte price to charge against the provided bandwidth
price = "1000000udvpn"
@@ this is the ideal price for 1GB and it is expressed in udvpn @@

# Address of the provider the node wants to operate under
provider = ""

# Public URL of the node
remote_url = "https://ip_node:tcp_port"
@@ example: https://123.456.78.90:12345 @@

[qos]# Limit max number of concurrent peers
max_peers = 250
```

Initialize the WireGuard configuration
```bash
docker run --rm \
    --volume ${HOME}/.sentinelnode:/root/.sentinelnode \
    sentinel-dvpn-node process wireguard config init
```

Open the file wireguard.toml
```bash
nano /root/.sentinelnode/wireguard.toml
```

Take note of the UDP port
```diff
# Name of the network interface
interface = "wg0"

# Port number to accept the incoming connections
listen_port = 54321
@@ this is the UDP port, you can change it and write it down as you need it later @@

# Server private key
private_key = "WEkdSO6cax3Sbo08mwmMyd2X617usVeVDTK/hdkfOmI="
```

Add an account key (the one in config.toml file)
```bash
docker run --rm \
    --interactive \
    --tty \
    --volume ${HOME}/.sentinelnode:/root/.sentinelnode \
    sentinel-dvpn-node process keys add
```

if you already have a mnemonic and simply want to recover your account, type this
```bash
docker run --rm \
    --interactive \
    --tty \
    --volume ${HOME}/.sentinelnode:/root/.sentinelnode \
    sentinel-dvpn-node process keys add --recover
```

It will be shown you an operator address (sent1), a node address (sentnode1) and a mnemonic which you will write down and store in a safe place.
Also, **send some DVPN tokens to the operator address** before starting it (50 DVPNs are enough)

Move created TLS keys
```bash
mv ${HOME}/tls.crt ${HOME}/.sentinelnode/tls.crt && \
mv ${HOME}/tls.key ${HOME}/.sentinelnode/tls.key

sudo chown root:root ${HOME}/.sentinelnode/tls.crt && \
sudo chown root:root ${HOME}/.sentinelnode/tls.key
```

Enable TCP port on Firewall (check file config.toml)
```bash
sudo ufw allow 12345/tcp
```

Enable UDP port on Firewall (check file wireguard.toml)
```bash
sudo ufw allow 54321/udp
```

Run the node (add your tcp and udp chosen ports).
```bash
docker run -d \
    --name sentinel-dvpn-node \
    --restart unless-stopped \
    --volume ${HOME}/.sentinelnode:/root/.sentinelnode \
    --volume /lib/modules:/lib/modules \
    --cap-drop ALL \
    --cap-add NET_ADMIN \
    --cap-add NET_BIND_SERVICE \
    --cap-add NET_RAW \
    --cap-add SYS_MODULE \
    --sysctl net.ipv4.ip_forward=1 \
    --sysctl net.ipv6.conf.all.disable_ipv6=0 \
    --sysctl net.ipv6.conf.all.forwarding=1 \
    --sysctl net.ipv6.conf.default.forwarding=1 \
    --publish 12345:12345/tcp \
    --publish 54321:54321/udp \
    sentinel-dvpn-node process start
```
Remember that autorestart function is enabled, this will prevent you to manually restart the node in case of problems.

## Node Management

#### This a list of commands to manage your node

Container list/details
```bash
docker ps -a
```

Node logs (you can change the number of logs with another value)
```bash
docker logs -f -n 100 sentinel-dvpn-node
```

Stop a node
```bash
docker stop sentinel-dvpn-node 
```

Start a stopped node
```bash
docker start sentinel-dvpn-node 
```

Remove a node
```bash
docker rm -f sentinel-dvpn-node
```
