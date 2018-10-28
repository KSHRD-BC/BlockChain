# Setup Your Environment
### Utility Packages
### Docker
Docker is a tool for deploying, executing, and managing containers. Hyperledger Fabric is by default packaged as a set of Docker images and it is ready to be run as Docker container.

To install the Docker, we can go to [Docker website](https://docs.docker.com/install/linux/docker-ce/ubuntu/):

**SETUP THE REPOSITORY**
```sh
# 1. Update the apt package index:
sudo apt-get update

# 2. Install packages to allow apt to use repository over HTTPS:
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

# 3. Add Docker's official GPG key:
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo apt-key fingerprint 0EBFCD88

# 4. Setup the stable repository.
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

**INSTALL THE DOCKER CE**
```sh
# 1. Update the apt package index
sudo apt-get update

# 2. Install the latest version of Docker CE
sudo apt-get install docker-ce
```

### Docker Compose

Docker Compose is a tool for defining and running multi-container Docker applications. This is the case of Hyperledger Fabric default setup.

Docker Compose is typically installed as a part of your Docker installation. If not, it is necessary to install it separately. Run ```docker-compose --version``` command to find out if Docker Compose is present on your system.

To install Docker Compose on your Ubuntu 16.04 LTS system, please follow the instructions describe below or [follow the instruction how to install the docker compose in docker website](https://docs.docker.com/compose/install/#install-compose):

```sh
sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

# Check the Docker Compose version
docker-compose --version
```

Add docker to current user group
```sh
sudo usermod -a -G docker $USER
```
**After adding docker to current user group, please logout and login again.**

### Go language

In this tutorial, we use the Go language as the basis of chaincode in Hyperledger Fabric. For Hyperledger Fabric, the Go language version is 1.7 or higher.

To install the Go language in the system, please follow the below instructions:

```sh
# Downloading the golang file
wget https://dl.google.com/go/go1.9.3.linux-amd64.tar.gz

# Extract it in home directory
tar -xvf go1.9.3.linux-amd64.tar.gz
```
Create a gopath directory in the **HOME directory**.
``` sh
mkdir $HOME/gopath
```

Then we need to set the **Environment Variables** for our Golang in ~/.bashrc file such ```GOPATH```, ```GOROOT```, or ```GOBIN```.

```sh
# set GOPATH and GOROOT
export GOPATH=$HOME/gopath
export GOROOT=$HOME/go
export GOBIN=$HOME/go/bin
export PATH=$PATH:/$GOROOT/bin
```

Then run the command ```source ~/.bashrc``` to build the environment variables in the **.bashrc** file
