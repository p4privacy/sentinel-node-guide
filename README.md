# <img src="https://user-images.githubusercontent.com/114076168/191721379-88f4b6ca-6463-4458-aab4-73d29d1bc7a0.jpg" width="35" height="35"> Sentinel Guide: How to Install a Node

How to install a Sentinel node for newbies

## Menu

* [Preliminary Operations](#preliminary-operations)
* [Install Docker](#install-docker)
* [Preparing the Docker Image](#preparing-the-docker-image)

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
