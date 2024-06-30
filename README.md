# cilium_srv6_lab

-- HowTo Prepare Ubuntu 24 for containerlab
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
