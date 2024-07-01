# Kubernetes with Cilium and Segmentrouting IPv6 on Containerlab

## Table of Contents

- [About](#about)
- [High Level Overview](#HLD)
- [Getting Started](#getting_started)
- [Usage](#usage)
- [Contributing](../CONTRIBUTING.md)

## About <a name = "about"></a>

Building a lab enviroment with two Kubernetes cluster, interconnected via SRv6. The enviroment is based on containerlab.

## High Level Overview = "HLD"></a>

![Hig Level Overview](https://github.com/juergenhofer-SC/cilium_srv6_lab/blob/main/K8s_SRv6_LAB_HLD.png)

## Getting Started <a name = "getting_started"></a>

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See [deployment](#deployment) for notes on how to deploy the project on a live system.

### Prerequisites

**HowTo Prepare Ubuntu 24 for containerlab**
 1. Install containerlab with docker
	- curl -sL <https://containerlab.dev/setup> | sudo bash -s "all"
 2. Add user and perms for Docker
	- sudo groupadd docker
	- sudo usermod -aG docker $MYUSER
 3. Verify Docker installation
	- docker run hello-world
 4. Install kind
	- [ $(uname -m) = x86_64 ] && curl -Lo ./kind <https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64> cmod +x ./kind
 5. Install Helm
	- curl <https://baltocdn.com/helm/signing.asc> | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
	- sudo apt-get install apt-transport-https --yes
	- echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] 		<https://baltocdn.com/helm/stable/debian/> all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list

**HowTo prepare for Cisco XRd**
 1. Download Cisco image form internal repo
     - docker login $local.artifactory
	- docker pull $local.artifactory/nso/ios-xr/xrd-control-plane:7.10.2

### Installing

A step by step series of examples that tell you how to get a development env running.

Say what the step will be

```
Give the example
```

And repeat

```
until finished
```

End with an example of getting some data out of the system or using it for a little demo.

## Usage <a name = "usage"></a>

Add notes about how to use the system.
