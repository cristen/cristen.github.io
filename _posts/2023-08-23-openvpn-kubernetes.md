---
layout: post
title: OpenVPN behind Kubernetes
date: 2023-08-23 16:24
category: DevOps
tags: ["networking", "kubernetes", "k8s", "vpn"]
---

# Introduction

At some point in my development I had for a project to create a sidecar in Kubernetes (K8s) that would 
act as a proxy. As such it would sit between the client (=other pods) and external machines (=internet).

Here's a list of commands I created to do so. The machine used was using Ubuntu 16.04.

```console
apt update
apt install -y openvpn
apt install -y wget curl unzip
cd /etc/openvpn
wget https://downloads.nordcdn.com/configs/archives/servers/ovpn.zip
unzip ovpn.zip
rm ovpn.zip
```
> this part installs openvpn on the machine

```console
touch credentials.txt
echo "login" > credentials.txt
echo "password" >> credentials.txt
```
> This credentials file is used to login to openVPN. The first line is the login and the second one is the password.

```console
mkdir -p /dev/net
mknod /dev/net/tun c 10 200
```
> mknod is a unix command to create devices on the machine. Here we create a device file and more specifically a TUN/TAP interface which can be used to implement a virtual private network for instance.

```console
openvpn --config "/etc/openvpn/ovpn_udp/"`ls /etc/openvpn/ovpn_udp/ | grep us | xargs shuf -n1 -e` --auth-user-pass /etc/openvpn/credentials.txt

curl https://api.nordvpn.com/vpn/check/full
```
> The first part of this command lists the openvpn list of servers and chooses one server in the US randomly. It then applies the configuration and uses our authfile to authenticate to the server. The curl part then checks that we are indeed behind the VPN.

## Checking that everything works well

If you want to check your external IP address you can use the following snippet:

```console
dig +short myip.opendns.com @resolver4.opendns.com
```

That's it!