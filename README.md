# Kubernetes with Cilium and Segmentrouting IPv6 on Containerlab

## Table of Contents

- [About](#about)
- [Getting Started](#getting_started)
- [Usage](#usage)
- [Contributing](../CONTRIBUTING.md)

## About <a name = "about"></a>

Building a lab enviroment with two Kubernetes cluster, interconnected via SRv6. The enviroemnt is based on containerlab.

## Getting Started <a name = "getting_started"></a>

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See [deployment](#deployment) for notes on how to deploy the project on a live system.

### Prerequisites

**HowTo Prepare Ubuntu 24 for containerlab**

 1. Install containerlab with docker
	1.1 curl -sL <https://containerlab.dev/setup> | sudo bash -s "all"
 2. Add user and perms for Docker
	 2.1 sudo groupadd docker
	 2.2 sudo usermod -aG docker $MYUSER
 3. Verify Docker installation
	 3.1 docker run hello-world
 4. Install kind
	 4.1 [ $(uname -m) = x86_64 ] && curl -Lo ./kind <https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64> cmod +x ./kind
 5. Install Helm
	 5.1 curl <https://baltocdn.com/helm/signing.asc> | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
	 5.1 sudo apt-get install apt-transport-https --yes
	 5.2 echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] 		<https://baltocdn.com/helm/stable/debian/> all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list

**HowTo prepare for Cisco XRd**
 
1. Download Cisco image form internal repo
	1.1 docker login $local.artifactory
	1.2 docker pull $local.artifactory/nso/ios-xr/xrd-control-plane:7.10.2

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

- HowTo Prepare Ubuntu 24 for containerlab
2. Install containerlab with docker
     a. curl -sL https://containerlab.dev/setup | sudo bash -s "all"
3. Add user and perms for Docker
     a. sudo groupadd docker
     b. sudo usermod -aG docker $MYUSER
4. Verify Docker installation
     a. docker run hello-world
5. Install kind
     a. [ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64 cmod +x ./kind
6. Install Helm
     a. curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
     b. sudo apt-get install apt-transport-https --yes
     c. echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list

-- HowTo prepare for Cisco XRd
1. Download Cisco image form internal repo
     a. docker login $local.artifactory
     b. docker pull $local.artifactory/nso/ios-xr/xrd-control-plane:7.10.2

