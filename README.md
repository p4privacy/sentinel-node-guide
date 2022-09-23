# <img src="https://user-images.githubusercontent.com/114076168/191721379-88f4b6ca-6463-4458-aab4-73d29d1bc7a0.jpg" width="35" height="35"> Sentinel Guide: How to Install a Node

How to install a Sentinel node for newbies

## Menu

* [Preliminary Operations](#preliminary-operations)
* [Install Docker](#install-docker)
* [Preparing the Docker Image](#preparing-the-docker-image)
* [Node Configuration](#node-configuration)

## Preliminary Operations

Update the list of available software packages
```bash
sudo apt update && sudo apt upgrade -y
```
Install Firewall
```bash
sudo apt-get install ufw
```
Allow Port 22
```bash
sudo ufw allow 22
```
Enable Port 22
```bash
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

Add user to Docker group
```bash
sudo usermod -aG docker $(whoami)
```

Reboot the machine
```bash
sudo reboot
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
Edit the configuration file ${HOME}/.sentinelnode/config.toml if required
```bash
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

# Name of the key with which to sign
from = "your_name"

[node]
# Time interval between each set_sessions operation
interval_set_sessions = "2m0s"

# Time interval between each update_sessions transaction
interval_update_sessions = "1h55m0s"

# Time interval between each set_status transaction
interval_update_status = "55m0s"

# API listen-address
listen_on = "0.0.0.0:12345"

# Name of the node
moniker = "Nome del vostro nodo che verrà visualizzato"

# Per Gigabyte price to charge against the provided bandwidth
price = "1000000udvpn"

# Address of the provider the node wants to operate under
provider = ""

# Public URL of the node
remote_url = "https://ip_node:tcp_port"

[qos]# Limit max number of concurrent peers
max_peers = 250
```
