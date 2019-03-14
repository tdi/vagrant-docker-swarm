#!/bin/bash
export DEBIAN_FRONTEND=noninteractive

#sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 \ --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
#echo "deb https://apt.dockerproject.org/repo ubuntu-trusty main" | sudo tee /etc/apt/sources.list.d/docker.list
#sudo apt-get update
## sudo apt-get install linux-image-extra-$(uname -r) linux-image-extra-virtual -y
#sudo apt-get install docker-engine --force-yes -y
#sudo usermod -aG docker vagrant
#sudo service docker start
#docker version

sudo apt-get update -y -qq

sudo apt-get install -y \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg-agent \
  software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sudo apt-get update -y -qq

#The following fails on docker-ce-cli, "package not found"
#sudo apt-get install -y docker-ce docker-ce-cli containerd.io

#It looks like this is all that's needed...maybe
sudo apt-get install -y docker-ce

sudo usermod -aG docker vagrant

#sudo service docker start


