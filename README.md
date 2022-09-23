# <img src="https://user-images.githubusercontent.com/114076168/191721379-88f4b6ca-6463-4458-aab4-73d29d1bc7a0.jpg" width="35" height="35"> Sentinel Node Guide

How to install a Sentinel node for newbies

## Menu

* [Install Docker](#install-docker)

## Install Docker

Update the list of available software packages

```bash
sudo apt-get update
```
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
