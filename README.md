# Tutorial 1: Hyperledger Fabric V1.2.0 - Create a Development Business Network on Ubuntu 16.04 LTS

This article is a tutorial that guides you how to create a Hyperledger Fabric v.1.2.0 business network on Ubuntu 16.04 LTS using the development tools that are found in the Hyperledger Fabric repository.

We will go through the process of setting up the Hyperledger Fabric prerequisites and later on we define and start Hyperledger Fabric blockchain network between three organizations.


- [Tutorial 1: Hyperledger Fabric V1.2.0 - Create a Development Business Network on Ubuntu 16.04 LTS](#tutorial-1-hyperledger-fabric-v120---create-a-development-business-network-on-ubuntu-1604-lts)
  - [Recommended Reading](#recommended-reading)
  - [Setup Your Environment](#setup-your-environment)
    - [Utility Packages](#utility-packages)
    - [Docker](#docker)
    - [Docker Compose](#docker-compose)
    - [Go language](#go-language)
  - [Retrieve Artifacts from Hyperledger Fabric Repsitories](#retrieve-artifacts-from-hyperledger-fabric-repsitories)
  - [Create Hyperledger Fabric Business Network](#create-hyperledger-fabric-business-network)
    - [Create Project Directory](#create-project-directory)
    - [Generate Peer and Orderer Certificates](#generate-peer-and-orderer-certificates)
    - [Create channel.tx and the Genesis Block Using the configtxgen Tool](#create-channeltx-and-the-genesis-block-using-the-configtxgen-tool)
      - [Creating/Modifying ```configtx.yaml```](#creatingmodifying-configtxyaml)
      - [Executing the configtxgen Tool](#executing-the-configtxgen-tool)
  - [Start the Hyperledger Fabric blockchain network](#start-the-hyperledger-fabric-blockchain-network)
    - [Modifying the docker-compose yaml Files](#modifying-the-docker-compose-yaml-files)
    - [Before running docker containers](#before-running-docker-containers)
    - [Start the docker Containers](#start-the-docker-containers)
    - [The channel](#the-channel)
      - [Create the channel](#create-the-channel)
      - [Join channel](#join-channel)
      - [Update anchor peers](#update-anchor-peers)
    - [Prepare chaincode](#prepare-chaincode)
    - [Install chaincode](#install-chaincode)
    - [Instantiate chaincode](#instantiate-chaincode)
    - [Execute tutorial scenario](#execute-tutorial-scenario)
      - [Query for initial state of the ledger](#query-for-initial-state-of-the-ledger)
      - [Invoke chaincode](#invoke-chaincode)
      - [Query for the new values](#query-for-the-new-values)
  - [Summary](#summary)
  - [What is Next?](#what-is-next)
  - [References?](#references)

## Recommended Reading
Before you start this tutorial, you may want to get familar with the basic concepts of Hyperledger Fabric. Official Hyperledger Fabric documentation provides comprehensive source of information related to Hyperledger Fabric configuration, modes of operation and prerequisites. We recommend to read the following articles and use them as the reference when going through this tutorial.

* Hyperledger Fabric Glossary - http://hyperledger-fabric.readthedocs.io/en/latest/glossary.html
* Hyperledger Fabric Model - http://hyperledger-fabric.readthedocs.io/en/latest/fabric_model.html
* Hyperledger Fabric Prerequisities - http://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html
* Hyperledger Fabric Samples - https://github.com/hyperledger/fabric-samples
* Building Your First Network - http://hyperledger-fabric.readthedocs.io/en/latest/build_network.html

## Setup Your Environment
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
sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose

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

## Retrieve Artifacts from Hyperledger Fabric Repsitories

To download and install **Hyperledger Fabric binaries** specific to your platform. This include downloading of ```cryptogen```, ```configtxgen```, ```configtxlator```, ```fabric-ca-client```, ```get-docker-images.sh```, ```orderer``` and ```peer``` tools and placing them into ```bin``` directory in the directory of your choice. In addition, the script will download Hyperlerdger Docker images into your local Docker registry.

Execute the following command to download Hyperledger Fabric binaries and Docker images:

```sh
curl -sSL http://bit.ly/2ysbOFE | bash -s <fabric-version> <fabric-ca-version> <third-parties>

curl -sSL http://bit.ly/2ysbOFE | bash -s 1.2.0 1.2.0 0.4.13
```

After run the command above, it will show following image. 

```sh
Installing hyperledger/fabric-samples repo

===> Checking out v1.2.0 branch of hyperledger/fabric-samples
HEAD is now at ed81d7b... [FAB-10811] fabric-ca sample is broken on v1.2

Installing Hyperledger Fabric binaries

===> Downloading version 1.2.0 platform specific fabric binaries
===> Downloading:  https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric/hyperledger-fabric/linux-amd64-1.2.0/hyperledger-fabric-linux-amd64-1.2.0.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 39.0M  100 39.0M    0     0  3683k      0  0:00:10  0:00:10 --:--:-- 4915k
==> Done.
===> Downloading version 1.2.0 platform specific fabric-ca-client binary
===> Downloading:  https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric-ca/hyperledger-fabric-ca/linux-amd64-1.2.0/hyperledger-fabric-ca-linux-amd64-1.2.0.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 4940k  100 4940k    0     0   956k      0  0:00:05  0:00:05 --:--:-- 1031k
==> Done.
.
.
.

===> List out hyperledger docker images
hyperledger/fabric-ca          1.2.0               66cc132bd09c        12 days ago         252MB
hyperledger/fabric-ca          latest              66cc132bd09c        12 days ago         252MB
hyperledger/fabric-tools       1.2.0               379602873003        12 days ago         1.51GB
hyperledger/fabric-tools       latest              379602873003        12 days ago         1.51GB
hyperledger/fabric-ccenv       1.2.0               6acf31e2d9a4        12 days ago         1.43GB
hyperledger/fabric-ccenv       latest              6acf31e2d9a4        12 days ago         1.43GB
hyperledger/fabric-orderer     1.2.0               4baf7789a8ec        12 days ago         152MB
hyperledger/fabric-orderer     latest              4baf7789a8ec        12 days ago         152MB
hyperledger/fabric-peer        1.2.0               82c262e65984        12 days ago         159MB
hyperledger/fabric-peer        latest              82c262e65984        12 days ago         159MB
hyperledger/fabric-zookeeper   0.4.10              2b51158f3898        2 weeks ago         1.44GB
hyperledger/fabric-zookeeper   latest              2b51158f3898        2 weeks ago         1.44GB
hyperledger/fabric-kafka       0.4.10              936aef6db0e6        2 weeks ago         1.45GB
hyperledger/fabric-kafka       latest              936aef6db0e6        2 weeks ago         1.45GB
hyperledger/fabric-couchdb     0.4.10              3092eca241fc        2 weeks ago         1.61GB
hyperledger/fabric-couchdb     latest              3092eca241fc        2 weeks ago         1.61GB
```
if docker.sock permission denied appear, enter following command:
```sh
~/bin$ chmod 777 /var/run/docker.sock

```


Then we can see the binary files and shell script file in the bin ```~/fabric-samples/bin``` directory
```sh
~/bin$ ll

total 131264
drwxrwxr-x  2 blockchain blockchain     4096 Jul  4 05:41 ./
drwxr-xr-x 22 blockchain blockchain     4096 Jul 16 14:39 ../
-rwxrwxr-x  1 blockchain blockchain 16784432 Jul  4 04:04 configtxgen*
-rwxrwxr-x  1 blockchain blockchain 17925784 Jul  4 04:04 configtxlator*
-rwxrwxr-x  1 blockchain blockchain  8660280 Jul  4 04:04 cryptogen*
-rwxrwxr-x  1 blockchain blockchain 17611704 Jul  4 04:04 discover*
-rwxrwxr-x  1 blockchain blockchain 14298688 Jul  4 05:41 fabric-ca-client*
-rwxrwxr-x  1 blockchain blockchain      817 Jul  4 04:04 get-docker-images.sh*
-rwxrwxr-x  1 blockchain blockchain  7152824 Jul  4 04:04 idemixgen*
-rwxrwxr-x  1 blockchain blockchain 22075240 Jul  4 04:04 orderer*
-rwxrwxr-x  1 blockchain blockchain 29880000 Jul  4 04:04 peer*
```

For your convenience, you can add this directory with Hyperledger Fabric binaries to your **PATH** environment variables. You can modify the **export PATH** line in your ~/.bashrc
```sh
# set GOPATH and GOROOT
export GOPATH=$HOME/gopath
export GOROOT=$HOME/go
export GOBIN=$HOME/go/bin
export PATH=$PATH:/$GOROOT/bin

# set Hyperledger Fabric
export PATH=$PATH:$HOME/fabric-samples/bin
```

And then run the command ```source ~/.bashrc``` to rebuild the export **PATH** environment variables.

## Create Hyperledger Fabric Business Network

This tutorial describes creating a business network and deploying chaincode using the Hyperledger Fabric v1.2.0 code base (found here: https://github.com/hyperledger/fabric.git). This tutorial exploits the Hyperledger Fabric v1.2.0 configuration toolset to create the business network consisting of the following items:
- Three orderer nodes (HRD Center Organization)
- OrdererType is Kafka (4 Kafka nodes and 3 zookeeper ensembles)
- Two peerOrganizations (Coocon and Webcash Company)
    - Coocon company has 2 peers
    - Webcash company has 2 peers
- Authentication Data Chaincode / Smart Contracts

At the end of this tutorial, you will have constructed a running instance of Hyperledger Fabric business network, as well as installed, instantiated, and executed chaincode.

For our tutorial we will make use of the Fabric deployment model based on Docker conatiners. Our network will run Docker containers running on our target Ubuntu 16.04 LTS (localhost).

### Create Project Directory

```sh
# Project path depends on your requirement
mkdir -p $GOPATH/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks
```

### Generate Peer and Orderer Certificates
Nodes (such as peers and orderers) are permitted to access business networks using a membership service provider, which is typically in the form of a certificate authority. In this example, we use the development tool named ```cryptogen``` to generate the required certificates. We use a local MSP to store the certs, which are essentially a local directory structure, for each peer and orderer. In production environments, you can exploit the ```fabric ca``` toolset introducing full-featured certificate authorities to generate the certificates.

```crytogen``` tool uses a ```yaml``` configuration file as its configuration - based on the content of this file, the required certificates are generated. We are going to create ```crypto-config.yaml``` file for our configuration. We are going to define two organizations of peers and three orderer organization.

Here is the listing of our ```crypto-config.yaml``` configuration file (for the purpose of simplicity, all comments are removed from this listing):

Reference: **Hyperledger Fabric Samples** [crypto-config.yaml](https://github.com/hyperledger/fabric-samples/blob/release-1.2/first-network/crypto-config.yaml)
```yaml
OrdererOrgs:
  # ---------------------------------------------------------------------------
  # Orderer === KSHRD CENTER
  # ---------------------------------------------------------------------------
  - Name: Orderer
    Domain: kshrd.com.kh
    Specs:
      - Hostname: orderer1
      - Hostname: orderer2
      - Hostname: orderer3
PeerOrgs:
  # ---------------------------------------------------------------------------
  # Coocon Organization
  # ---------------------------------------------------------------------------
  - Name: Coocon
    Domain: coocon.kshrd.com.kh
    Template:
      Count: 2
    Users:
      Count: 1

  # ---------------------------------------------------------------------------
  # Webcash Organization
  # ---------------------------------------------------------------------------
  - Name: Webcash
    Domain: webcash.kshrd.com.kh
    Template:
      Count: 2
    Users:
      Count: 1

```

To generate certificates, run the following command:

```sh
cryptogen generate --config=./crypto-config.yaml
```

After running of crytogen tool you should see the following output in console:
```sh
blockchain@DBNIS:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks$ cryptogen generate --config=./crypto-config.yaml

coocon.kshrd.com.kh
webcash.kshrd.com.kh
```

In addition, the new crypto-config directory has been created and contains various certificates and keys for orderers and peers.

First, let's check content under ```orderOrganizations```:
```sh
blockchain@DBNIS:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks/crypto-config/ordererOrganizations$ tree
.
`-- kshrd.com.kh
    |-- ca
    |   |-- 62d9b8330ddf6930c9120655079692775d79b09ddae5a6568a6755298f2bb18f_sk
    |   |-- ca.kshrd.com.kh-cert.pem
    |   |-- cd840b07f566494fb94f7318c21a03a1a2ebdc5ec53f4b214e7dbb6af4f65392_sk
    |   `-- fa7912fb28f41a4ec3bff0af537a919a61de0af869652b1df60e3cf7a1b981e0_sk
    |-- msp
    |   |-- admincerts
    |   |   |-- Admin@kshrd.com.kh-cert.pem
    |   |   `-- ca.kshrd.com.kh-cert.pem
    |   |-- cacerts
    |   |   `-- ca.kshrd.com.kh-cert.pem
    |   `-- tlscacerts
    |       `-- tlsca.kshrd.com.kh-cert.pem
    |-- orderers
    |   |-- orderer1.kshrd.com.kh
    |   |   |-- msp
    |   |   |   |-- admincerts
    |   |   |   |   `-- Admin@kshrd.com.kh-cert.pem
    |   |   |   |-- cacerts
    |   |   |   |   `-- ca.kshrd.com.kh-cert.pem
    |   |   |   |-- keystore
    |   |   |   |   `-- e8b1c30347faebf85570b716fef89b5ecf888993ae6418c0f0bf24fe72968529_sk
    |   |   |   |-- signcerts
    |   |   |   |   `-- orderer1.kshrd.com.kh-cert.pem
    |   |   |   `-- tlscacerts
    |   |   |       `-- tlsca.kshrd.com.kh-cert.pem
    |   |   `-- tls
    |   |       |-- ca.crt
    |   |       |-- server.crt
    |   |       `-- server.key
    |   |-- orderer2.kshrd.com.kh
    |   |   |-- msp
    |   |   |   |-- admincerts
    |   |   |   |   `-- Admin@kshrd.com.kh-cert.pem
    |   |   |   |-- cacerts
    |   |   |   |   `-- ca.kshrd.com.kh-cert.pem
    |   |   |   |-- keystore
    |   |   |   |   `-- f10af6a64885e3216743387b5aa8bfd40be612a2af5ff756ce4d18c8024157ca_sk
    |   |   |   |-- signcerts
    |   |   |   |   `-- orderer2.kshrd.com.kh-cert.pem
    |   |   |   `-- tlscacerts
    |   |   |       `-- tlsca.kshrd.com.kh-cert.pem
    |   |   `-- tls
    |   |       |-- ca.crt
    |   |       |-- server.crt
    |   |       `-- server.key
    |   `-- orderer3.kshrd.com.kh
    |       |-- msp
    |       |   |-- admincerts
    |       |   |   `-- Admin@kshrd.com.kh-cert.pem
    |       |   |-- cacerts
    |       |   |   `-- ca.kshrd.com.kh-cert.pem
    |       |   |-- keystore
    |       |   |   `-- 147cf5b13dd715fe4b77d06dfa8a22a157a186ff2e02212fb6a17ccf126fc2f6_sk
    |       |   |-- signcerts
    |       |   |   `-- orderer3.kshrd.com.kh-cert.pem
    |       |   `-- tlscacerts
    |       |       `-- tlsca.kshrd.com.kh-cert.pem
    |       `-- tls
    |           |-- ca.crt
    |           |-- server.crt
    |           `-- server.key
    |-- tlsca
    |   |-- 29d28b9aef50e504d3dee39af3f46ed7a633f8db88d9b08f3f266de9da107b52_sk
    |   |-- 52fd839effe3317c146b107754f956257aeb9314208696b0da463427f969ac9d_sk
    |   |-- 6259ef486a2f859c61c29384d956d4dc71a7e25d69d97c05a994444eee197938_sk
    |   `-- tlsca.kshrd.com.kh-cert.pem
    `-- users
        `-- Admin@kshrd.com.kh
            |-- msp
            |   |-- admincerts
            |   |   `-- Admin@kshrd.com.kh-cert.pem
            |   |-- cacerts
            |   |   `-- ca.kshrd.com.kh-cert.pem
            |   |-- keystore
            |   |   `-- 30781059ff0899fe3735a58776c452737adf6aa6ed73bf80acbccc5be8f60bc2_sk
            |   |-- signcerts
            |   |   `-- Admin@kshrd.com.kh-cert.pem
            |   `-- tlscacerts
            |       `-- tlsca.kshrd.com.kh-cert.pem
            `-- tls
                |-- ca.crt
                |-- client.crt
                `-- client.key

41 directories, 44 files
```

Second, let's check content under ```peerOrganizations```:
```sh
blockchain@DBNIS:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks/crypto-config/peerOrganizations$ tree
.
|-- coocon.kshrd.com.kh
|   |-- ca
|   |   |-- 5585e44510645d290b1f5437db9947627f0fb4dff4e82243c1861b5c6ec0d9e5_sk
|   |   |-- 64a1ff19bb4a6ef0e854c0f7bcaccdc7bb9970dab7099943f1255d2cf1d16c9c_sk
|   |   |-- b9213a2e7d864ca45f52f7e9f56db6183334f414ebcadf2c751bba2c243c4969_sk
|   |   `-- ca.coocon.kshrd.com.kh-cert.pem
|   |-- msp
|   |   |-- admincerts
|   |   |   |-- Admin@coocon.kshrd.com.kh-cert.pem
|   |   |   `-- ca.coocon.kshrd.com.kh-cert.pem
|   |   |-- cacerts
|   |   |   `-- ca.coocon.kshrd.com.kh-cert.pem
|   |   `-- tlscacerts
|   |       `-- tlsca.coocon.kshrd.com.kh-cert.pem
|   |-- peers
|   |   |-- peer0.coocon.kshrd.com.kh
|   |   |   |-- msp
|   |   |   |   |-- admincerts
|   |   |   |   |   `-- Admin@coocon.kshrd.com.kh-cert.pem
|   |   |   |   |-- cacerts
|   |   |   |   |   `-- ca.coocon.kshrd.com.kh-cert.pem
|   |   |   |   |-- keystore
|   |   |   |   |   `-- 74376e2ddf2fd97ff56a1d8c761d1ab9817ec20b35805860e1cdbdacb3244685_sk
|   |   |   |   |-- signcerts
|   |   |   |   |   `-- peer0.coocon.kshrd.com.kh-cert.pem
|   |   |   |   `-- tlscacerts
|   |   |   |       `-- tlsca.coocon.kshrd.com.kh-cert.pem
|   |   |   `-- tls
|   |   |       |-- ca.crt
|   |   |       |-- server.crt
|   |   |       `-- server.key
|   |   `-- peer1.coocon.kshrd.com.kh
|   |       |-- msp
|   |       |   |-- admincerts
|   |       |   |   `-- Admin@coocon.kshrd.com.kh-cert.pem
|   |       |   |-- cacerts
|   |       |   |   `-- ca.coocon.kshrd.com.kh-cert.pem
|   |       |   |-- keystore
|   |       |   |   `-- 7848a3104a3df4c7906d69a2772038599edc2e21e51c53c719295453530e46d2_sk
|   |       |   |-- signcerts
|   |       |   |   `-- peer1.coocon.kshrd.com.kh-cert.pem
|   |       |   `-- tlscacerts
|   |       |       `-- tlsca.coocon.kshrd.com.kh-cert.pem
|   |       `-- tls
|   |           |-- ca.crt
|   |           |-- server.crt
|   |           `-- server.key
|   |-- tlsca
|   |   |-- 1fcebf2e6851b192ceb4dc57942d57ad3ace0de74a78421a39fa3dbbf938d546_sk
|   |   |-- 40250c6c2d89b306a979ae490d3ae1e8ee4117327b45214e19519a9f06f0963f_sk
|   |   |-- b39f3decd1c964e6b92b8ebcf54a54278ea5d338beacb46871105344737a8c80_sk
|   |   `-- tlsca.coocon.kshrd.com.kh-cert.pem
|   `-- users
|       |-- Admin@coocon.kshrd.com.kh
|       |   |-- msp
|       |   |   |-- admincerts
|       |   |   |   `-- Admin@coocon.kshrd.com.kh-cert.pem
|       |   |   |-- cacerts
|       |   |   |   `-- ca.coocon.kshrd.com.kh-cert.pem
|       |   |   |-- keystore
|       |   |   |   `-- 8c4c15f75f1f9d50a6726b227a18fe3797a8005171eb801874485bd78f774ae0_sk
|       |   |   |-- signcerts
|       |   |   |   `-- Admin@coocon.kshrd.com.kh-cert.pem
|       |   |   `-- tlscacerts
|       |   |       `-- tlsca.coocon.kshrd.com.kh-cert.pem
|       |   `-- tls
|       |       |-- ca.crt
|       |       |-- client.crt
|       |       `-- client.key
|       `-- User1@coocon.kshrd.com.kh
|           |-- msp
|           |   |-- admincerts
|           |   |   `-- User1@coocon.kshrd.com.kh-cert.pem
|           |   |-- cacerts
|           |   |   `-- ca.coocon.kshrd.com.kh-cert.pem
|           |   |-- keystore
|           |   |   `-- aadad041cd0af7d976d72d90da58785b9bbad45320b4f9bf30e90959942f211c_sk
|           |   |-- signcerts
|           |   |   `-- User1@coocon.kshrd.com.kh-cert.pem
|           |   `-- tlscacerts
|           |       `-- tlsca.coocon.kshrd.com.kh-cert.pem
|           `-- tls
|               |-- ca.crt
|               |-- client.crt
|               `-- client.key
`-- webcash.kshrd.com.kh
    |-- ca
    |   |-- 04386a844879f88aede01af0f901c9ca59c49d64f564f8f1b19f7d2ff63183d9_sk
    |   |-- 7933c140dfd5e460f04eccb464c634457888160fa9a791114df6128ac84db31c_sk
    |   |-- ca.webcash.kshrd.com.kh-cert.pem
    |   `-- f867041e19a3ec72f6e744559fcedfefb422c211455f379822d7dcbb40ee8e27_sk
    |-- msp
    |   |-- admincerts
    |   |   |-- Admin@webcash.kshrd.com.kh-cert.pem
    |   |   `-- ca.webcash.kshrd.com.kh-cert.pem
    |   |-- cacerts
    |   |   `-- ca.webcash.kshrd.com.kh-cert.pem
    |   `-- tlscacerts
    |       `-- tlsca.webcash.kshrd.com.kh-cert.pem
    |-- peers
    |   |-- peer0.webcash.kshrd.com.kh
    |   |   |-- msp
    |   |   |   |-- admincerts
    |   |   |   |   `-- Admin@webcash.kshrd.com.kh-cert.pem
    |   |   |   |-- cacerts
    |   |   |   |   `-- ca.webcash.kshrd.com.kh-cert.pem
    |   |   |   |-- keystore
    |   |   |   |   `-- 787a09b8a395b700ae58b40fb8ac62574f998f6fa95c35cf2e8623b61d2d7460_sk
    |   |   |   |-- signcerts
    |   |   |   |   `-- peer0.webcash.kshrd.com.kh-cert.pem
    |   |   |   `-- tlscacerts
    |   |   |       `-- tlsca.webcash.kshrd.com.kh-cert.pem
    |   |   `-- tls
    |   |       |-- ca.crt
    |   |       |-- server.crt
    |   |       `-- server.key
    |   `-- peer1.webcash.kshrd.com.kh
    |       |-- msp
    |       |   |-- admincerts
    |       |   |   `-- Admin@webcash.kshrd.com.kh-cert.pem
    |       |   |-- cacerts
    |       |   |   `-- ca.webcash.kshrd.com.kh-cert.pem
    |       |   |-- keystore
    |       |   |   `-- dc7be07bba9a2b193d6a8d5b57401f87a253c44d5c3d83c7d30c384413984a3b_sk
    |       |   |-- signcerts
    |       |   |   `-- peer1.webcash.kshrd.com.kh-cert.pem
    |       |   `-- tlscacerts
    |       |       `-- tlsca.webcash.kshrd.com.kh-cert.pem
    |       `-- tls
    |           |-- ca.crt
    |           |-- server.crt
    |           `-- server.key
    |-- tlsca
    |   |-- 6fe1ac929388e2a8ff64858b2b2837d67d7b40908e85b30e0f16177f34af129e_sk
    |   |-- 905f70691b37f21958c10225e14868bc2acfad640ccc0974df598c3e5fb18e6f_sk
    |   |-- cf1f9c966cb1a11a8f338dd8755db369d9faefa9aa8519f9e5b4e45a3e76d8e0_sk
    |   `-- tlsca.webcash.kshrd.com.kh-cert.pem
    `-- users
        |-- Admin@webcash.kshrd.com.kh
        |   |-- msp
        |   |   |-- admincerts
        |   |   |   `-- Admin@webcash.kshrd.com.kh-cert.pem
        |   |   |-- cacerts
        |   |   |   `-- ca.webcash.kshrd.com.kh-cert.pem
        |   |   |-- keystore
        |   |   |   `-- 7a9a5f3c71d30751d7450ffb31759dacf8e966d302132fb9e1f79693c916210b_sk
        |   |   |-- signcerts
        |   |   |   `-- Admin@webcash.kshrd.com.kh-cert.pem
        |   |   `-- tlscacerts
        |   |       `-- tlsca.webcash.kshrd.com.kh-cert.pem
        |   `-- tls
        |       |-- ca.crt
        |       |-- client.crt
        |       `-- client.key
        `-- User1@webcash.kshrd.com.kh
            |-- msp
            |   |-- admincerts
            |   |   `-- User1@webcash.kshrd.com.kh-cert.pem
            |   |-- cacerts
            |   |   `-- ca.webcash.kshrd.com.kh-cert.pem
            |   |-- keystore
            |   |   `-- c18035ac2ac47aaa0e52f8ad5ac60d921154438e8cfc798d3cb9ae12e2ab9117_sk
            |   |-- signcerts
            |   |   `-- User1@webcash.kshrd.com.kh-cert.pem
            |   `-- tlscacerts
            |       `-- tlsca.webcash.kshrd.com.kh-cert.pem
            `-- tls
                |-- ca.crt
                |-- client.crt
                `-- client.key

82 directories, 88 files
```

### Create channel.tx and the Genesis Block Using the configtxgen Tool

Now we generated the certificates and keys, we can now configure the ```configtx.yaml``` file. This yaml file serves as input to the ```configtxgen``` tool and generates the following important artifacts such as:
- **channel.tx**

The channel creation transaction. This transaction lets you create the Hyperledger Fabric channel. The channel is the location where the ledger exists and the mechanism that lets peers join business networks.
- **Genesis Block**

The Genesis block is the first block in our blockchain. It is used to bootstrap the ordering service and holds the channel configuration.

- **Anchor peers transactions**

The anchor peer transactions specify each Org's Anchor Peer on this channel for communicating from one organization to other one.

#### Creating/Modifying ```configtx.yaml```
We recommend that you can start with the existing ```configtx.yaml``` in the ```fabric-sample``` project in the github as it contains a template that requires only minor modifications for our needs.

The ```configtx.yaml``` file is broken into several sections:

```Profile```:  Profiles describe the organization structure of your network.

```Organization```: The details regarding individual organizations.

```Orderer```: The details regarding the Orderer parameters.

```Application```: Application defaults - not needed for this tutorial.

The following example shows a fully configured ```configtx.yaml``` file (for the purpose of simplicity, all comments are removed from this listing):

Reference: **Hyperledger Fabric Samples** [configtx.yaml](https://github.com/hyperledger/fabric-samples/blob/release-1.2/first-network/configtx.yaml)

```yaml
################################################################################
#   Section: Organizations
################################################################################
Organizations:
    - &OrdererOrg
        Name: OrdererOrg
        ID: OrdererMSP
        MSPDir: crypto-config/ordererOrganizations/kshrd.com.kh/msp

    - &Coocon
        Name: CooconMSP
        ID: CooconMSP
        MSPDir: crypto-config/peerOrganizations/coocon.kshrd.com.kh/msp
        AnchorPeers:
            - Host: peer0.coocon.kshrd.com.kh
              Port: 7051

    - &Webcash
        Name: WebcashMSP
        ID: WebcashMSP
        MSPDir: crypto-config/peerOrganizations/webcash.kshrd.com.kh/msp
        AnchorPeers:
            - Host: peer0.webcash.kshrd.com.kh
              Port: 7051

################################################################################
#   SECTION: Capabilities
################################################################################
Capabilities:
    Global: &ChannelCapabilities
        V1_1: true
    Orderer: &OrdererCapabilities
        V1_1: true
    Application: &ApplicationCapabilities
        V1_2: true

################################################################################
#   SECTION: Application
################################################################################
Application: &ApplicationDefaults
    Organizations:

################################################################################
#   SECTION: Orderer
################################################################################
Orderer: &OrdererDefaults
    OrdererType: kafka
    Addresses:
        - orderer1.kshrd.com.kh:7050
        - orderer2.kshrd.com.kh:7050
        - orderer3.kshrd.com.kh:7050
    BatchTimeout: 2s
    BatchSize:
        MaxMessageCount: 10
        AbsoluteMaxBytes: 99 MB
        PreferredMaxBytes: 512 KB
    Kafka:
        Brokers:
            - kafka1:9092
            - kafka2:9092
            - kafka3:9092
            - kafka4:9092
    Organizations:

################################################################################
#   Profile
################################################################################
Profiles:
    KSHRDCooconWebcashOrdererGenesis:
        Capabilities:
            <<: *ChannelCapabilities
        Orderer:
            <<: *OrdererDefaults
            OrdererType: kafka
            Addresses:
                - orderer1.kshrd.com.kh:7050
                - orderer2.kshrd.com.kh:7050
                - orderer3.kshrd.com.kh:7050
            Organizations:
                - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Consortiums:
            CooconWebConsortium:
                Organizations:
                    - *Coocon
                    - *Webcash

    CooconWebcashOrgChannel:
        Consortium: CooconWebConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Coocon
                - *Webcash
            Capabilities:
                <<: *ApplicationCapabilities
```

You can review the file or can modify it as necessary. However, the following items are key modifications:
- The organizations that we specified in the profiles section are named exactly as we named them in the ```cryptogen``` tool and its ```crypto-config.yaml``` configuration file.
- We modified the ID and Name fields to append MSP for the peers.
- We modified the MSPDir to point to the output directories from the ```cryptogen tool```.

#### Executing the configtxgen Tool
To create orderer genesis block, run the following commands:

```sh
# Create the config directory to store the channel-artifacts
mkdir config

# Generate the Genesis Block
configtxgen -profile KSHRDCooconWebcashOrdererGenesis -outputBlock ./config/genesis.block
```

Output:
```sh
blockchain@DBNIS:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks$ configtxgen -profile KSHRDCooconWebcashOrdererGenesis -outputBlock ./config/genesis.block

2018-07-17 15:11:17.833 KST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-07-17 15:11:17.839 KST [common/tools/configtxgen] doOutputBlock -> INFO 002 Generating genesis block
2018-07-17 15:11:17.840 KST [common/tools/configtxgen] doOutputBlock -> INFO 003 Writing genesis block
```

After we created the orderer genesis block it is a time to create channel configuration transaction.

```sh
# Generate channel configuration transaction
configtxgen -profile CooconWebcashOrgChannel -outputCreateChannelTx ./config/channel.tx -channelID mychannel
```

```sh
blockchain@DBNIS:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks$ configtxgen -profile CooconWebcashOrgChannel -outputCreateChannelTx ./config/channel.tx -channelID mychannel

2018-07-17 15:14:17.289 KST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-07-17 15:14:17.294 KST [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
2018-07-17 15:14:17.296 KST [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 003 Writing new channel tx
```

The last operation we are going to perform with ```configtxgen``` is the definition of anchor peers for our organizations. This is especially important if there are more peers belonging to a single organization.

Run the following two commands to define anchor peers for each organization. Note that the ```asOrg``` parameter refers to the MSP ID definitions in ```configtx.yaml```.

```sh
# Generate anchor peer transaction for Coocon Organization
configtxgen -profile CooconWebcashOrgChannel -outputAnchorPeersUpdate ./config/CooconMSPanchors.tx -channelID mychannel -asOrg CooconMSP

# generate anchor peer transaction for Webcash Organization
configtxgen -profile CooconWebcashOrgChannel -outputAnchorPeersUpdate ./config/WebcashMSPanchors.tx -channelID mychannel -asOrg WebcashMSP
```

```sh
blockchain@DBNIS:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks$ configtxgen -profile CooconWebcashOrgChannel -outputAnchorPeersUpdate ./config/CooconMSPanchors.tx -channelID mychannel -asOrg CooconMSP
2018-07-17 15:16:15.543 KST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-07-17 15:16:15.548 KST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2018-07-17 15:16:15.548 KST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update

blockchain@DBNIS:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks$ configtxgen -profile CooconWebcashOrgChannel -outputAnchorPeersUpdate ./config/WebcashMSPanchors.tx -channelID mychannel -asOrg WebcashMSP
2018-07-17 15:16:44.683 KST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-07-17 15:16:44.689 KST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2018-07-17 15:16:44.689 KST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update
```

List files generated by `configtxgen`

```sh
blockchain@DBNIS:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks/config$ ll
total 32
drwxrwxr-x 2 blockchain blockchain  4096 Jul 17 15:16 ./
drwxrwxr-x 4 blockchain blockchain  4096 Jul 17 15:11 ../
-rw-r--r-- 1 blockchain blockchain   292 Jul 17 15:16 CooconMSPanchors.tx
-rw-r--r-- 1 blockchain blockchain   295 Jul 17 15:16 WebcashMSPanchors.tx
-rw-r--r-- 1 blockchain blockchain   359 Jul 17 15:14 channel.tx
-rw-r--r-- 1 blockchain blockchain 11812 Jul 17 15:11 genesis.block
```

## Start the Hyperledger Fabric blockchain network
To start our network we will use ```docker-compose``` tool. Based on its configuration, we launch containers based on the Docker images we downloaded in the beginning.
### Modifying the docker-compose yaml Files
```docker-compose``` tools is using yaml configuration files where various aspects of the containers and their network connection are defined. You can start with configuration yaml file from scratch or leverage yaml configuration from the "first-network" example.

Inside your network directory, you should have a directory structure similar to the following:

<img src="https://github.com/PENHCHET/hyperledger-fabric-tutorials/raw/master/assets/.1.%20Hyperledger%20Fabric%20v1.2%20-%20Create%20a%20Development%20Business%20Network%20on%20Ubuntu%2016.04%20LTS_images/82030bed.png" />

We have to create the .env file in current directory. There are many environment variables for using with the Docker Compose file.

```sh
COMPOSE_PROJECT_NAME=net
```

The ```docker-compose.yaml``` content showed by the following:

Reference: **Hyperledger Fabric Samples** [docker-compose.yaml](https://github.com/hyperledger/fabric-samples/blob/release-1.2/basic-network/docker-compose.yml)

```yaml
#
# Copyright IBM Corp All Rights Reserved
#
# SPDX-License-Identifier: Apache-2.0
#
version: '2'

networks:
  basic:

services:
  ca.coocon.kshrd.com.kh:
    image: hyperledger/fabric-ca:1.2.0
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca.coocon.kshrd.com.kh
      - FABRIC_CA_SERVER_CA_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.coocon.kshrd.com.kh-cert.pem
      - FABRIC_CA_SERVER_CA_KEYFILE=/etc/hyperledger/fabric-ca-server-config/c0f7ea66b8810f163943ec204fe0188f4703563e568c41796e7ca39ded563524_sk
    ports:
      - "7054:7054"
    command: sh -c 'fabric-ca-server start -b admin:admin -d'
    volumes:
      - ./crypto-config/peerOrganizations/coocon.kshrd.com.kh/ca/:/etc/hyperledger/fabric-ca-server-config
    container_name: ca.coocon.kshrd.com.kh
    networks:
      - basic

  ca.webcash.kshrd.com.kh:
    image: hyperledger/fabric-ca:1.2.0
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca.webcash.kshrd.com.kh
      - FABRIC_CA_SERVER_CA_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.webcash.kshrd.com.kh-cert.pem
      - FABRIC_CA_SERVER_CA_KEYFILE=/etc/hyperledger/fabric-ca-server-config/cc31a08512e1e64dcc63c57de7cc146ebde693bf9df967f3e3c8ac89160e9e7b_sk
    ports:
      - "8054:7054"
    command: sh -c 'fabric-ca-server start -b admin:admin -d'
    volumes:
      - ./crypto-config/peerOrganizations/webcash.kshrd.com.kh/ca/:/etc/hyperledger/fabric-ca-server-config
    container_name: ca.webcash.kshrd.com.kh
    networks:
      - basic

  zookeeper1:
    image: hyperledger/fabric-zookeeper
    container_name: zookeeper1
    restart: always
    environment:
      - ZOO_MY_ID=1
      - ZOO_SERVERS=server.1=zookeeper1:2888:3888 server.2=zookeeper2:2888:3888 server.3=zookeeper3:2888:3888
    ports:
      - 2181
      - 2888
      - 3888
    networks:
      - basic

  zookeeper2:
    image: hyperledger/fabric-zookeeper
    container_name: zookeeper2
    restart: always
    environment:
      - ZOO_MY_ID=2
      - ZOO_SERVERS=server.1=zookeeper1:2888:3888 server.2=zookeeper2:2888:3888 server.3=zookeeper3:2888:3888
    ports:
      - 2181
      - 2888
      - 3888
    networks:
      - basic

  zookeeper3:
    image: hyperledger/fabric-zookeeper
    container_name: zookeeper3
    restart: always
    environment:
      - ZOO_MY_ID=3
      - ZOO_SERVERS=server.1=zookeeper1:2888:3888 server.2=zookeeper2:2888:3888 server.3=zookeeper3:2888:3888
    ports:
      - 2181
      - 2888
      - 3888
    networks:
      - basic

  kafka1:
    image: hyperledger/fabric-kafka
    container_name: kafka1
#        restart: always
    environment:
      - KAFKA_ADVERTISED_HOST_NAME=kafka1
      - KAFKA_ADVERTISED_PORT=9092
      - KAFKA_BROKER_ID=1
      - KAFKA_MESSAGE_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
      - KAFKA_REPLICA_FETCH_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
      - KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE=false
      - KAFKA_NUM_REPLICA_FETCHERS=1
      - KAFKA_DEFAULT_REPLICATION_FACTOR=3
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
    ports:
      - 9092
    depends_on:
      - zookeeper1
      - zookeeper2
      - zookeeper3
    networks:
      - basic

  kafka2:
    image: hyperledger/fabric-kafka
    container_name: kafka2
#        restart: always
    environment:
      - KAFKA_ADVERTISED_HOST_NAME=kafka2
      - KAFKA_ADVERTISED_PORT=9092
      - KAFKA_BROKER_ID=2
      - KAFKA_MESSAGE_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
      - KAFKA_REPLICA_FETCH_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
      - KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE=false
      - KAFKA_NUM_REPLICA_FETCHERS=1
      - KAFKA_DEFAULT_REPLICATION_FACTOR=3
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
    ports:
      - 9092
    depends_on:
      - zookeeper1
      - zookeeper2
      - zookeeper3
    networks:
      - basic

  kafka3:
    image: hyperledger/fabric-kafka
    container_name: kafka3
#        restart: always
    environment:
      - KAFKA_ADVERTISED_HOST_NAME=kafka3
      - KAFKA_ADVERTISED_PORT=9092
      - KAFKA_BROKER_ID=3
      - KAFKA_MESSAGE_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
      - KAFKA_REPLICA_FETCH_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
      - KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE=false
      - KAFKA_NUM_REPLICA_FETCHERS=1
      - KAFKA_DEFAULT_REPLICATION_FACTOR=3
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
    ports:
      - 9092
    depends_on:
      - zookeeper1
      - zookeeper2
      - zookeeper3
    networks:
      - basic

  kafka4:
    image: hyperledger/fabric-kafka
    container_name: kafka4
#        restart: always
    environment:
      - KAFKA_ADVERTISED_HOST_NAME=kafka4
      - KAFKA_ADVERTISED_PORT=9092
      - KAFKA_BROKER_ID=4
      - KAFKA_MESSAGE_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
      - KAFKA_REPLICA_FETCH_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
      - KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE=false
      - KAFKA_NUM_REPLICA_FETCHERS=1
      - KAFKA_DEFAULT_REPLICATION_FACTOR=2
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
    ports:
      - 9092
    depends_on:
      - zookeeper1
      - zookeeper2
      - zookeeper3
    networks:
      - basic

  orderer1.kshrd.com.kh:
    container_name: orderer1.kshrd.com.kh
    image: hyperledger/fabric-orderer:1.2.0
    environment:
      - ORDERER_HOST=orderer1.kshrd.com.kh
      - ORDERER_GENERAL_LOGLEVEL=debug
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_LISTENPORT=7050
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/etc/hyperledger/configtx/genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/etc/hyperledger/msp/orderer/msp
      # Kafka Orderer Type
      - CONFIGTX_ORDERER_BATCHTIMEOUT=1s
      - CONFIGTX_ORDERER_ORDERERTYPE=kafka
      - CONFIGTX_ORDERER_KAFKA_BROKERS=[kafka1:9092,kafka2:9092,kafka3:9092,kafka4:9092]
      - ORDERER_KAFKA_RETRY_SHORTINTERVAL=1s
      - ORDERER_KAFKA_RETRY_SHORTTOTAL=30s
      - ORDERER_KAFKA_VERBOSE=true
      # - ORDERER_GENERAL_GENESISPROFILE=SampleInsecureKafka
      # - ORDERER_ABSOLUTEMAXBYTES=${ORDERER_ABSOLUTEMAXBYTES}
      # - ORDERER_PREFERREDMAXBYTES=${ORDERER_PREFERREDMAXBYTES}
      - ORDERER_ABSOLUTEMAXBYTES=10 MB
      - ORDERER_PREFERREDMAXBYTES=512 KB

      # enabled TLS
      # - ORDERER_GENERAL_TLS_ENABLED=true
      # - ORDERER_GENERAL_TLS_PRIVATEKEY=/etc/hyperledger/orderer/tls/server.key
      # - ORDERER_GENERAL_TLS_CERTIFICATE=/etc/hyperledger/orderer/tls/server.crt
      # - ORDERER_GENERAL_TLS_ROOTCAS=[/etc/hyperledger/orderer/tls/ca.crt]
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/orderer
    command: orderer
    ports:
      - 7050:7050
    volumes:
      - ./config/:/etc/hyperledger/configtx
      - ./crypto-config/ordererOrganizations/kshrd.com.kh/orderers/orderer1.kshrd.com.kh/:/etc/hyperledger/msp/orderer
    networks:
      - basic
    depends_on:
      - kafka1
      - kafka2
      - kafka3
      - kafka4

  orderer2.kshrd.com.kh:
    container_name: orderer2.kshrd.com.kh
    image: hyperledger/fabric-orderer:1.2.0
    environment:
      - ORDERER_HOST=orderer2.kshrd.com.kh
      - ORDERER_GENERAL_LOGLEVEL=debug
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_LISTENPORT=7050
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/etc/hyperledger/configtx/genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/etc/hyperledger/msp/orderer/msp
      # Kafka Orderer Type
      - CONFIGTX_ORDERER_BATCHTIMEOUT=1s
      - CONFIGTX_ORDERER_ORDERERTYPE=kafka
      - CONFIGTX_ORDERER_KAFKA_BROKERS=[kafka1:9092,kafka2:9092,kafka3:9092,kafka4:9092]
      - ORDERER_KAFKA_RETRY_SHORTINTERVAL=1s
      - ORDERER_KAFKA_RETRY_SHORTTOTAL=30s
      - ORDERER_KAFKA_VERBOSE=true
      # - ORDERER_GENERAL_GENESISPROFILE=SampleInsecureKafka
      # - ORDERER_ABSOLUTEMAXBYTES=${ORDERER_ABSOLUTEMAXBYTES}
      # - ORDERER_PREFERREDMAXBYTES=${ORDERER_PREFERREDMAXBYTES}
      - ORDERER_ABSOLUTEMAXBYTES=10 MB
      - ORDERER_PREFERREDMAXBYTES=512 KB

      # enabled TLS
      # - ORDERER_GENERAL_TLS_ENABLED=true
      # - ORDERER_GENERAL_TLS_PRIVATEKEY=/etc/hyperledger/orderer/tls/server.key
      # - ORDERER_GENERAL_TLS_CERTIFICATE=/etc/hyperledger/orderer/tls/server.crt
      # - ORDERER_GENERAL_TLS_ROOTCAS=[/etc/hyperledger/orderer/tls/ca.crt]
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/orderer
    command: orderer
    ports:
      - 8050:7050
    volumes:
      - ./config/:/etc/hyperledger/configtx
      - ./crypto-config/ordererOrganizations/kshrd.com.kh/orderers/orderer2.kshrd.com.kh/:/etc/hyperledger/msp/orderer
    networks:
      - basic
    depends_on:
      - kafka1
      - kafka2
      - kafka3
      - kafka4

  orderer3.kshrd.com.kh:
    container_name: orderer3.kshrd.com.kh
    image: hyperledger/fabric-orderer:1.2.0
    environment:
      - ORDERER_HOST=orderer3.kshrd.com.kh
      - ORDERER_GENERAL_LOGLEVEL=debug
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_LISTENPORT=7050
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/etc/hyperledger/configtx/genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/etc/hyperledger/msp/orderer/msp
      # Kafka Orderer Type
      - CONFIGTX_ORDERER_BATCHTIMEOUT=1s
      - CONFIGTX_ORDERER_ORDERERTYPE=kafka
      - CONFIGTX_ORDERER_KAFKA_BROKERS=[kafka1:9092,kafka2:9092,kafka3:9092,kafka4:9092]
      - ORDERER_KAFKA_RETRY_SHORTINTERVAL=1s
      - ORDERER_KAFKA_RETRY_SHORTTOTAL=30s
      - ORDERER_KAFKA_VERBOSE=true
      # - ORDERER_GENERAL_GENESISPROFILE=SampleInsecureKafka
      # - ORDERER_ABSOLUTEMAXBYTES=${ORDERER_ABSOLUTEMAXBYTES}
      # - ORDERER_PREFERREDMAXBYTES=${ORDERER_PREFERREDMAXBYTES}
      - ORDERER_ABSOLUTEMAXBYTES=10 MB
      - ORDERER_PREFERREDMAXBYTES=512 KB

      # enabled TLS
      # - ORDERER_GENERAL_TLS_ENABLED=true
      # - ORDERER_GENERAL_TLS_PRIVATEKEY=/etc/hyperledger/orderer/tls/server.key
      # - ORDERER_GENERAL_TLS_CERTIFICATE=/etc/hyperledger/orderer/tls/server.crt
      # - ORDERER_GENERAL_TLS_ROOTCAS=[/etc/hyperledger/orderer/tls/ca.crt]
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/orderer
    command: orderer
    ports:
      - 9050:7050
    volumes:
      - ./config/:/etc/hyperledger/configtx
      - ./crypto-config/ordererOrganizations/kshrd.com.kh/orderers/orderer3.kshrd.com.kh/:/etc/hyperledger/msp/orderer
    networks:
      - basic
    depends_on:
      - kafka1
      - kafka2
      - kafka3
      - kafka4

  peer0.coocon.kshrd.com.kh:
    container_name: peer0.coocon.kshrd.com.kh
    image: hyperledger/fabric-peer:1.2.0
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_PEER_ID=peer0.coocon.kshrd.com.kh
      - CORE_LOGGING_PEER=debug
      - CORE_CHAINCODE_LOGGING_LEVEL=DEBUG
      - CORE_PEER_LOCALMSPID=CooconMSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/peer/
      - CORE_PEER_ADDRESS=peer0.coocon.kshrd.com.kh:7051
      # # the following setting starts chaincode containers on the same
      # # bridge network as the peers
      # # https://docs.docker.com/compose/networking/
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_basic
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb0.coocon.kshrd.com.kh:5984
      # The CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME and CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD
      # provide the credentials for ledger to connect to CouchDB.  The username and password must
      # match the username and password set for the associated CouchDB.
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: peer node start
    # command: peer node start --peer-chaincodedev=true
    ports:
      - 7051:7051
      - 7052:7052
      - 7053:7053
    volumes:
      - /var/run/:/host/var/run/
      - ./crypto-config/peerOrganizations/coocon.kshrd.com.kh/peers/peer0.coocon.kshrd.com.kh/msp:/etc/hyperledger/msp/peer
      - ./crypto-config/peerOrganizations/coocon.kshrd.com.kh/users:/etc/hyperledger/msp/users
      - ./config:/etc/hyperledger/configtx
    depends_on:
      - couchdb0.coocon.kshrd.com.kh
    networks:
      - basic

  couchdb0.coocon.kshrd.com.kh:
    container_name: couchdb0.coocon.kshrd.com.kh
    image: hyperledger/fabric-couchdb
    # Populate the COUCHDB_USER and COUCHDB_PASSWORD to set an admin user and password
    # for CouchDB.  This will prevent CouchDB from operating in an "Admin Party" mode.
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    ports:
      - 5984:5984
    networks:
      - basic

  peer1.coocon.kshrd.com.kh:
    container_name: peer1.coocon.kshrd.com.kh
    image: hyperledger/fabric-peer:1.2.0
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_PEER_ID=peer1.coocon.kshrd.com.kh
      - CORE_LOGGING_PEER=debug
      - CORE_CHAINCODE_LOGGING_LEVEL=DEBUG
      - CORE_PEER_LOCALMSPID=CooconMSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/peer/
      - CORE_PEER_ADDRESS=peer1.coocon.kshrd.com.kh:7051
      # # the following setting starts chaincode containers on the same
      # # bridge network as the peers
      # # https://docs.docker.com/compose/networking/
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_basic
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb1.coocon.kshrd.com.kh:5984
      # The CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME and CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD
      # provide the credentials for ledger to connect to CouchDB.  The username and password must
      # match the username and password set for the associated CouchDB.
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: peer node start
    # command: peer node start --peer-chaincodedev=true
    ports:
      - 8051:7051
      - 8052:7052
      - 8053:7053
    volumes:
      - /var/run/:/host/var/run/
      - ./crypto-config/peerOrganizations/coocon.kshrd.com.kh/peers/peer1.coocon.kshrd.com.kh/msp:/etc/hyperledger/msp/peer
      - ./crypto-config/peerOrganizations/coocon.kshrd.com.kh/users:/etc/hyperledger/msp/users
      - ./config:/etc/hyperledger/configtx
    depends_on:
      - couchdb1.coocon.kshrd.com.kh
    networks:
      - basic

  couchdb1.coocon.kshrd.com.kh:
    container_name: couchdb1.coocon.kshrd.com.kh
    image: hyperledger/fabric-couchdb
    # Populate the COUCHDB_USER and COUCHDB_PASSWORD to set an admin user and password
    # for CouchDB.  This will prevent CouchDB from operating in an "Admin Party" mode.
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    ports:
      - 6984:5984
    networks:
      - basic


  peer0.webcash.kshrd.com.kh:
    container_name: peer0.webcash.kshrd.com.kh
    image: hyperledger/fabric-peer:1.2.0
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_PEER_ID=peer0.webcash.kshrd.com.kh
      - CORE_LOGGING_PEER=debug
      - CORE_CHAINCODE_LOGGING_LEVEL=DEBUG
      - CORE_PEER_LOCALMSPID=WebcashMSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/peer/
      - CORE_PEER_ADDRESS=peer0.webcash.kshrd.com.kh:7051
      # # the following setting starts chaincode containers on the same
      # # bridge network as the peers
      # # https://docs.docker.com/compose/networking/
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_basic
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb0.webcash.kshrd.com.kh:5984
      # The CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME and CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD
      # provide the credentials for ledger to connect to CouchDB.  The username and password must
      # match the username and password set for the associated CouchDB.
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: peer node start
    # command: peer node start --peer-chaincodedev=true
    ports:
      - 9051:7051
      - 9052:7052
      - 9053:7053
    volumes:
      - /var/run/:/host/var/run/
      - ./crypto-config/peerOrganizations/webcash.kshrd.com.kh/peers/peer0.webcash.kshrd.com.kh/msp:/etc/hyperledger/msp/peer
      - ./crypto-config/peerOrganizations/webcash.kshrd.com.kh/users:/etc/hyperledger/msp/users
      - ./config:/etc/hyperledger/configtx
    depends_on:
      - couchdb0.webcash.kshrd.com.kh
    networks:
      - basic

  couchdb0.webcash.kshrd.com.kh:
    container_name: couchdb0.webcash.kshrd.com.kh
    image: hyperledger/fabric-couchdb
    # Populate the COUCHDB_USER and COUCHDB_PASSWORD to set an admin user and password
    # for CouchDB.  This will prevent CouchDB from operating in an "Admin Party" mode.
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    ports:
      - 7984:5984
    networks:
      - basic

  peer1.webcash.kshrd.com.kh:
    container_name: peer1.webcash.kshrd.com.kh
    image: hyperledger/fabric-peer:1.2.0
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_PEER_ID=peer1.webcash.kshrd.com.kh
      - CORE_LOGGING_PEER=debug
      - CORE_CHAINCODE_LOGGING_LEVEL=DEBUG
      - CORE_PEER_LOCALMSPID=WebcashMSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/peer/
      - CORE_PEER_ADDRESS=peer1.webcash.kshrd.com.kh:7051
      # # the following setting starts chaincode containers on the same
      # # bridge network as the peers
      # # https://docs.docker.com/compose/networking/
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_basic
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb1.webcash.kshrd.com.kh:5984
      # The CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME and CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD
      # provide the credentials for ledger to connect to CouchDB.  The username and password must
      # match the username and password set for the associated CouchDB.
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: peer node start
    # command: peer node start --peer-chaincodedev=true
    ports:
      - 10051:7051
      - 10052:7052
      - 10053:7053
    volumes:
      - /var/run/:/host/var/run/
      - ./crypto-config/peerOrganizations/webcash.kshrd.com.kh/peers/peer1.webcash.kshrd.com.kh/msp:/etc/hyperledger/msp/peer
      - ./crypto-config/peerOrganizations/webcash.kshrd.com.kh/users:/etc/hyperledger/msp/users
      - ./config:/etc/hyperledger/configtx
    depends_on:
      - couchdb1.webcash.kshrd.com.kh
    networks:
      - basic

  couchdb1.webcash.kshrd.com.kh:
    container_name: couchdb1.webcash.kshrd.com.kh
    image: hyperledger/fabric-couchdb
    # Populate the COUCHDB_USER and COUCHDB_PASSWORD to set an admin user and password
    # for CouchDB.  This will prevent CouchDB from operating in an "Admin Party" mode.
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    ports:
      - 8984:5984
    networks:
      - basic

  cli:
    container_name: cli
    image: hyperledger/fabric-tools
    tty: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_LOGGING_LEVEL=DEBUG
      - CORE_PEER_ID=cli  
      - CORE_PEER_ADDRESS=peer0.coocon.kshrd.com.kh:7051
      - CORE_PEER_LOCALMSPID=CooconMSP
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/users/Admin@coocon.kshrd.com.kh/msp
      - CORE_CHAINCODE_KEEPALIVE=10
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:
      - /var/run/:/host/var/run/
      - ./chaincodes/go:/opt/gopath/src/github.com/kshrdsmartcontract/go
      - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
      - ./config:/etc/hyperledger/configtx
    networks:
      - basic
    depends_on:
      # Orderer Organization HRD Center
      - orderer1.kshrd.com.kh
      - orderer2.kshrd.com.kh
      - orderer3.kshrd.com.kh
      # Coocon Organization
      - peer0.coocon.kshrd.com.kh
      - couchdb0.coocon.kshrd.com.kh
      - peer1.coocon.kshrd.com.kh
      - couchdb1.coocon.kshrd.com.kh
      # Webcash Organization
      - peer0.webcash.kshrd.com.kh
      - couchdb0.webcash.kshrd.com.kh
      - peer1.webcash.kshrd.com.kh
      - couchdb1.webcash.kshrd.com.kh
```


### Before running docker containers  
We need to change CA Key File for each Organization's CA inside `docker-composer.yaml`

<img src="https://github.com/PENHCHET/hyperledger-fabric-tutorials/raw/master/assets/.1.%20Hyperledger%20Fabric%20v1.2%20-%20Create%20a%20Development%20Business%20Network%20on%20Ubuntu%2016.04%20LTS_images/change_CA_Keyfile.png" />

CA KeyFile can be found in `cryto-config/peerOrganizations` folder for each orginazation

<img src="https://github.com/PENHCHET/hyperledger-fabric-tutorials/raw/master/assets/.1.%20Hyperledger%20Fabric%20v1.2%20-%20Create%20a%20Development%20Business%20Network%20on%20Ubuntu%2016.04%20LTS_images/coocon_ca_keyfile.png" />

<img src="https://github.com/PENHCHET/hyperledger-fabric-tutorials/raw/master/assets/.1.%20Hyperledger%20Fabric%20v1.2%20-%20Create%20a%20Development%20Business%20Network%20on%20Ubuntu%2016.04%20LTS_images/webcash_ca_keyfile.png" />


### Start the docker Containers

After we have generated the certificates, the genesis block, the channel transaction configuration, and created or modified the appropriate yaml files, we are read to start our network. use the following command to start the network.
```sh
# -d = Run the docker container in background
docker-compose -f docker-compose.yaml up -d

# Or the Default file of the docker-compose is docker-compose.yaml so we don't need to specify the file name

docker-compose -f docker-compose.yaml up
```

```sh
blockchain@DBNIS:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks$ docker-compose -f docker-compose.yaml up -d

WARNING: The COMPOSE_PROJECT_NAME variable is not set. Defaulting to a blank string.
Creating ca.coocon.kshrd.com.kh        ... done
Creating zookeeper2                    ... done
Creating ca.webcash.kshrd.com.kh       ... done
Creating zookeeper1                    ... done
Creating zookeeper3                    ... done
Creating couchdb1.coocon.kshrd.com.kh  ... done
Creating couchdb0.coocon.kshrd.com.kh  ... done
Creating couchdb1.webcash.kshrd.com.kh ... done
Creating couchdb0.webcash.kshrd.com.kh ... done
Creating peer0.coocon.kshrd.com.kh     ... done
Creating peer1.coocon.kshrd.com.kh     ... done
Creating peer1.webcash.kshrd.com.kh    ... done
Creating peer0.webcash.kshrd.com.kh    ... done
Creating kafka1                        ... done
Creating kafka2                        ... done
Creating kafka3                        ... done
Creating kafka4                        ... done
Creating orderer3.kshrd.com.kh         ... done
Creating orderer2.kshrd.com.kh         ... done
Creating orderer1.kshrd.com.kh         ... done
Creating cli                           ... done
```

You can use ```docker ps``` command to list the execusting containers. In our case, you should see the following containers executing:

- 2 peers in each organization
- 1 couchDB in each peer
- 3 orderer nodes
- 4 Kafka nodes
- 3 Zookeeper nodes
- 1 CLI

After running ```docker-compose``` and ```docker ps``` you can expect output similar to the following:
```sh
blockchain@DBNIS:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks$ docker ps

CONTAINER ID        IMAGE                              COMMAND                  CREATED             STATUS              PORTS                                                                       NAMES
5d6c53943337        hyperledger/fabric-tools           "/bin/bash"              41 seconds ago      Up 40 seconds                                                                                   cli
7c188d7f36fa        hyperledger/fabric-orderer:1.2.0   "orderer"                43 seconds ago      Up 41 seconds       0.0.0.0:7050->7050/tcp                                                      orderer1.kshrd.com.kh
4c9f015b2476        hyperledger/fabric-orderer:1.2.0   "orderer"                43 seconds ago      Up 42 seconds       0.0.0.0:9050->7050/tcp                                                      orderer3.kshrd.com.kh
e995b5305c3c        hyperledger/fabric-orderer:1.2.0   "orderer"                43 seconds ago      Up 41 seconds       0.0.0.0:8050->7050/tcp                                                      orderer2.kshrd.com.kh
0a669300e023        hyperledger/fabric-kafka           "/docker-entrypoint.…"   46 seconds ago      Up 44 seconds       9093/tcp, 0.0.0.0:32789->9092/tcp                                           kafka2
916f26282ff2        hyperledger/fabric-kafka           "/docker-entrypoint.…"   46 seconds ago      Up 44 seconds       9093/tcp, 0.0.0.0:32788->9092/tcp                                           kafka3
a20645f638bc        hyperledger/fabric-kafka           "/docker-entrypoint.…"   46 seconds ago      Up 44 seconds       9093/tcp, 0.0.0.0:32787->9092/tcp                                           kafka4
d5cdef32f2e0        hyperledger/fabric-kafka           "/docker-entrypoint.…"   46 seconds ago      Up 43 seconds       9093/tcp, 0.0.0.0:32786->9092/tcp                                           kafka1
70cc4181314b        hyperledger/fabric-peer:1.2.0      "peer node start"        48 seconds ago      Up 44 seconds       0.0.0.0:9051->7051/tcp, 0.0.0.0:9052->7052/tcp, 0.0.0.0:9053->7053/tcp      peer0.webcash.kshrd.com.kh
762c88f37391        hyperledger/fabric-peer:1.2.0      "peer node start"        48 seconds ago      Up 46 seconds       0.0.0.0:10051->7051/tcp, 0.0.0.0:10052->7052/tcp, 0.0.0.0:10053->7053/tcp   peer1.webcash.kshrd.com.kh
b461dc86c048        hyperledger/fabric-peer:1.2.0      "peer node start"        50 seconds ago      Up 45 seconds       0.0.0.0:8051->7051/tcp, 0.0.0.0:8052->7052/tcp, 0.0.0.0:8053->7053/tcp      peer1.coocon.kshrd.com.kh
5abe633f883e        hyperledger/fabric-peer:1.2.0      "peer node start"        51 seconds ago      Up 45 seconds       0.0.0.0:7051-7053->7051-7053/tcp                                            peer0.coocon.kshrd.com.kh
7f742ad960c1        hyperledger/fabric-couchdb         "tini -- /docker-ent…"   52 seconds ago      Up 48 seconds       4369/tcp, 9100/tcp, 0.0.0.0:7984->5984/tcp                                  couchdb0.webcash.kshrd.com.kh
696d93e60dc0        hyperledger/fabric-couchdb         "tini -- /docker-ent…"   52 seconds ago      Up 48 seconds       4369/tcp, 9100/tcp, 0.0.0.0:8984->5984/tcp                                  couchdb1.webcash.kshrd.com.kh
0058fa35c697        hyperledger/fabric-couchdb         "tini -- /docker-ent…"   52 seconds ago      Up 51 seconds       4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp                                  couchdb0.coocon.kshrd.com.kh
21c55fcf7c81        hyperledger/fabric-couchdb         "tini -- /docker-ent…"   52 seconds ago      Up 50 seconds       4369/tcp, 9100/tcp, 0.0.0.0:6984->5984/tcp                                  couchdb1.coocon.kshrd.com.kh
37088af844a5        hyperledger/fabric-zookeeper       "/docker-entrypoint.…"   52 seconds ago      Up 46 seconds       0.0.0.0:32785->2181/tcp, 0.0.0.0:32784->2888/tcp, 0.0.0.0:32782->3888/tcp   zookeeper3
a96bce1046d0        hyperledger/fabric-zookeeper       "/docker-entrypoint.…"   52 seconds ago      Up 48 seconds       0.0.0.0:32783->2181/tcp, 0.0.0.0:32781->2888/tcp, 0.0.0.0:32780->3888/tcp   zookeeper1
f471f732ddaa        hyperledger/fabric-ca:1.2.0        "sh -c 'fabric-ca-se…"   52 seconds ago      Up 51 seconds       0.0.0.0:8054->7054/tcp                                                      ca.webcash.kshrd.com.kh
6888989a165e        hyperledger/fabric-zookeeper       "/docker-entrypoint.…"   52 seconds ago      Up 51 seconds       0.0.0.0:32779->2181/tcp, 0.0.0.0:32778->2888/tcp, 0.0.0.0:32777->3888/tcp   zookeeper2
83e29338776f        hyperledger/fabric-ca:1.2.0        "sh -c 'fabric-ca-se…"   52 seconds ago      Up 50 seconds       0.0.0.0:7054->7054/tcp                                                      ca.coocon.kshrd.com.kh
```

### The channel

After the Docker container start, we can use the **Command Line Interface (CLI)** or **Fabric SDK** to interact with the blockchain network. In this tutorial we use the **CLI container** to interact with the blockchain network. We can enter to the **CLI container** by using the following command:

```sh
docker exec -it cli bash
```

```sh
blockchain@DBNIS:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks$ docker exec -it cli bash
root@5d6c53943337:/opt/gopath/src/github.com/hyperledger/fabric/peer#
```

After you enter the CLI container, you can interact with the other peers by prefixing our peer commands with the appropriate environment variables. Usually, this means pointing to the certificates for that peer. You don't need to do that when interacting with peer0 of Coocon organization which is set as the default one in CLI container.

Here is the list of environment variables that need to be used as the prefix for peer commands when interacting with peer0 of Coocon organization, peer1 of Coocon organization, peer0 of Webcash organization and peer1 of Webcash organization respectively.

```sh
# Peer0 of Coocon organization
CORE_PEER_LOCALMSPID=CooconMSP
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/users/Admin@coocon.kshrd.com.kh/msp
CORE_PEER_ADDRESS=peer0.coocon.kshrd.com.kh:7051
```

```sh
# Peer1 of Coocon organization
CORE_PEER_LOCALMSPID=CooconMSP
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/users/Admin@coocon.kshrd.com.kh/msp
CORE_PEER_ADDRESS=peer1.coocon.kshrd.com.kh:7051
```

```sh
# Peer0 of Webcash organization
CORE_PEER_LOCALMSPID=WebcashMSP
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/webcash.kshrd.com.kh/users/Admin@webcash.kshrd.com.kh/msp
CORE_PEER_ADDRESS=peer0.webcash.kshrd.com.kh:7051
```

```sh
# Peer1 of Webcash organization
CORE_PEER_LOCALMSPID=WebcashMSP
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/webcash.kshrd.com.kh/users/Admin@webcash.kshrd.com.kh/msp
CORE_PEER_ADDRESS=peer1.webcash.kshrd.com.kh:7051
```

#### Create the channel

The first command that we issue is the ```peer create channel``` command. This command targets the orderer (where the channels must be created) and uses the ```channel.tx``` and the channel name that is created using the ```configtxgen``` tool. To create the channel, run the following command to create the channel:

```sh
peer channel create -o orderer1.kshrd.com.kh:7050 -o orderer2.kshrd.com.kh:7050 -o orderer3.kshrd.com.kh:7050 -c mychannel -f /etc/hyperledger/configtx/channel.tx
```

``` bash
root@f034fb307bfc:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel create -o orderer1.kshrd.com.kh:7050 -o orderer2.kshrd.com.kh:7050 -o orderer3.kshrd.com.kh:7050 -c mychannel -f /etc/hyperledger/configtx/channel.tx
```

The ```peer channel create``` command returns a genesis block which will be used to join the channel. You can check that by running ```ll``` command to review that the file ```mychannel.block``` has been created.

```sh
root@5d6c53943337:/opt/gopath/src/github.com/hyperledger/fabric/peer# ll
total 28
drwxr-xr-x 3 root root  4096 Jul 17 06:34 ./
drwxr-xr-x 3 root root  4096 Jul 17 06:32 ../
drwxr-xr-x 4 1000 1000  4096 Jul 17 06:28 crypto/
-rw-r--r-- 1 root root 12475 Jul 17 06:34 mychannel.block
```

#### Join channel

After the orderer creates the channel, we can have the peers join the channel, again using the peer CLI:

Join **peer0.coocon.kshrd.com.kh** to the channel.
```
# Join peer0.coocon.kshrd.com.kh to the channel.
CORE_PEER_LOCALMSPID=CooconMSP
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/users/Admin@coocon.kshrd.com.kh/msp
CORE_PEER_ADDRESS=peer0.coocon.kshrd.com.kh:7051

# To join the channel
peer channel join -b mychannel.block
```

Join **peer1.coocon.kshrd.com.kh** to the channel.
```
CORE_PEER_LOCALMSPID=CooconMSP
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/users/Admin@coocon.kshrd.com.kh/msp
CORE_PEER_ADDRESS=peer1.coocon.kshrd.com.kh:7051
peer channel join -b mychannel.block
```

Join **peer0.webcash.kshrd.com.kh** to the channel.
```sh
# Join peer0.webcash.kshrd.com.kh to the channel.
CORE_PEER_LOCALMSPID=WebcashMSP
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/webcash.kshrd.com.kh/users/Admin@webcash.kshrd.com.kh/msp
CORE_PEER_ADDRESS=peer0.webcash.kshrd.com.kh:7051
peer channel join -b mychannel.block
```

Join **peer1.webcash.kshrd.com.kh** to the channel.
```sh
# Join peer1.webcash.kshrd.com.kh to the channel.
CORE_PEER_LOCALMSPID=WebcashMSP
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/webcash.kshrd.com.kh/users/Admin@webcash.kshrd.com.kh/msp
CORE_PEER_ADDRESS=peer1.webcash.kshrd.com.kh:7051
peer channel join -b mychannel.block
```

#### Update anchor peers

To update Anchor Peer for Coocon Organization
```sh
# Set Environment Variable to Peer0 in Coocon Organization
CORE_PEER_LOCALMSPID=CooconMSP
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/users/Admin@coocon.kshrd.com.kh/msp
CORE_PEER_ADDRESS=peer0.coocon.kshrd.com.kh:7051

# update the Anchor Peer
peer channel update -o orderer1.kshrd.com.kh:7050 -o orderer2.kshrd.com.kh:7050 -o orderer3.kshrd.com.kh:7050 -c mychannel -f /etc/hyperledger/configtx/CooconMSPanchors.tx
```

To update Anchor Peer for Webcash Organization
```sh
# Set Environment Variable to Peer0 in Webcash Organization
CORE_PEER_LOCALMSPID=WebcashMSP
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/webcash.kshrd.com.kh/users/Admin@webcash.kshrd.com.kh/msp
CORE_PEER_ADDRESS=peer0.webcash.kshrd.com.kh:7051

# update the Anchor Peer
peer channel update -o orderer1.kshrd.com.kh:7050 -o orderer2.kshrd.com.kh:7050 -o orderer3.kshrd.com.kh:7050 -c mychannel -f /etc/hyperledger/configtx/WebcashMSPanchors.tx
```
Exit from CLI Bash
```sh
exit
```

### Prepare chaincode

```sh
mkdir -p chaincodes/go
```

Create kshrdsmartcontract.go(anyname) inside chaincodes/go directory.
```go
/*
Copyright IBM Corp. 2016 All Rights Reserved.
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
		 http://www.apache.org/licenses/LICENSE-2.0
Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/

package main

//WARNING - this chaincode's ID is hard-coded in chaincode_example04 to illustrate one way of
//calling chaincode from a chaincode. If this example is modified, chaincode_example04.go has
//to be modified as well with the new ID of chaincode_example02.
//chaincode_example05 show's how chaincode ID can be passed in as a parameter instead of
//hard-coding.

import (
	"fmt"
	"strconv"

	"github.com/hyperledger/fabric/core/chaincode/shim"
	pb "github.com/hyperledger/fabric/protos/peer"
)

// SimpleChaincode example simple Chaincode implementation
type SimpleChaincode struct {
}

func (t *SimpleChaincode) Init(stub shim.ChaincodeStubInterface) pb.Response {
	fmt.Println("ex02 Init")
	_, args := stub.GetFunctionAndParameters()
	var A, B string    // Entities
	var Aval, Bval int // Asset holdings
	var err error

	if len(args) != 4 {
		return shim.Error("Incorrect number of arguments. Expecting 4")
	}

	// Initialize the chaincode
	A = args[0]
	Aval, err = strconv.Atoi(args[1])
	if err != nil {
		return shim.Error("Expecting integer value for asset holding")
	}
	B = args[2]
	Bval, err = strconv.Atoi(args[3])
	if err != nil {
		return shim.Error("Expecting integer value for asset holding")
	}
	fmt.Printf("Aval = %d, Bval = %d\n", Aval, Bval)

	// Write the state to the ledger
	err = stub.PutState(A, []byte(strconv.Itoa(Aval)))
	if err != nil {
		return shim.Error(err.Error())
	}

	err = stub.PutState(B, []byte(strconv.Itoa(Bval)))
	if err != nil {
		return shim.Error(err.Error())
	}

	return shim.Success(nil)
}

func (t *SimpleChaincode) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
	fmt.Println("ex02 Invoke")
	function, args := stub.GetFunctionAndParameters()
	if function == "invoke" {
		// Make payment of X units from A to B
		return t.invoke(stub, args)
	} else if function == "delete" {
		// Deletes an entity from its state
		return t.delete(stub, args)
	} else if function == "query" {
		// the old "Query" is now implemtned in invoke
		return t.query(stub, args)
	}

	return shim.Error("Invalid invoke function name. Expecting \"invoke\" \"delete\" \"query\"")
}

// Transaction makes payment of X units from A to B
func (t *SimpleChaincode) invoke(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	var A, B string    // Entities
	var Aval, Bval int // Asset holdings
	var X int          // Transaction value
	var err error

	if len(args) != 3 {
		return shim.Error("Incorrect number of arguments. Expecting 3")
	}

	A = args[0]
	B = args[1]

	// Get the state from the ledger
	// TODO: will be nice to have a GetAllState call to ledger
	Avalbytes, err := stub.GetState(A)
	if err != nil {
		return shim.Error("Failed to get state")
	}
	if Avalbytes == nil {
		return shim.Error("Entity not found")
	}
	Aval, _ = strconv.Atoi(string(Avalbytes))

	Bvalbytes, err := stub.GetState(B)
	if err != nil {
		return shim.Error("Failed to get state")
	}
	if Bvalbytes == nil {
		return shim.Error("Entity not found")
	}
	Bval, _ = strconv.Atoi(string(Bvalbytes))

	// Perform the execution
	X, err = strconv.Atoi(args[2])
	if err != nil {
		return shim.Error("Invalid transaction amount, expecting a integer value")
	}
	Aval = Aval - X
	Bval = Bval + X
	fmt.Printf("Aval = %d, Bval = %d\n", Aval, Bval)

	// Write the state back to the ledger
	err = stub.PutState(A, []byte(strconv.Itoa(Aval)))
	if err != nil {
		return shim.Error(err.Error())
	}

	err = stub.PutState(B, []byte(strconv.Itoa(Bval)))
	if err != nil {
		return shim.Error(err.Error())
	}

	return shim.Success(nil)
}

// Deletes an entity from state
func (t *SimpleChaincode) delete(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	if len(args) != 1 {
		return shim.Error("Incorrect number of arguments. Expecting 1")
	}

	A := args[0]

	// Delete the key from the state in ledger
	err := stub.DelState(A)
	if err != nil {
		return shim.Error("Failed to delete state")
	}

	return shim.Success(nil)
}

// query callback representing the query of a chaincode
func (t *SimpleChaincode) query(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	var A string // Entities
	var err error

	if len(args) != 1 {
		return shim.Error("Incorrect number of arguments. Expecting name of the person to query")
	}

	A = args[0]

	// Get the state from the ledger
	Avalbytes, err := stub.GetState(A)
	if err != nil {
		jsonResp := "{\"Error\":\"Failed to get state for " + A + "\"}"
		return shim.Error(jsonResp)
	}

	if Avalbytes == nil {
		jsonResp := "{\"Error\":\"Nil amount for " + A + "\"}"
		return shim.Error(jsonResp)
	}

	jsonResp := "{\"Name\":\"" + A + "\",\"Amount\":\"" + string(Avalbytes) + "\"}"
	fmt.Printf("Query Response:%s\n", jsonResp)
	return shim.Success(Avalbytes)
}

func main() {
	err := shim.Start(new(SimpleChaincode))
	if err != nil {
		fmt.Printf("Error starting Simple chaincode: %s", err)
	}
}
```


### Install chaincode

Right now, we have fully configured and running Hyperledger Fabric network. However, the network does not contain any business logic - from this perspective, we can consider it as "empty" and as of now we have no way how to enter data there. Hyperledger Fabric blockchain applications interact with the ledger through the chaincode and its methods. so as a next step we need to install (deploy) chaincode on each peer.

We can install chaincode on each peer using the following command:

Install in **peer0.coocon.kshrd.com.kh**
```sh
docker exec -e "CORE_PEER_LOCALMSPID=CooconMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/users/Admin@coocon.kshrd.com.kh/msp" cli peer chaincode install -n kshrdsmartcontract -v 1.0 -p github.com/kshrdsmartcontract/go -l golang
```

Install in **peer1.coocon.kshrd.com.kh**
```sh
docker exec -e "CORE_PEER_LOCALMSPID=CooconMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/users/Admin@coocon.kshrd.com.kh/msp" -e "CORE_PEER_ADDRESS=peer1.coocon.kshrd.com.kh:7051" cli peer chaincode install -n kshrdsmartcontract -v 1.0 -p github.com/kshrdsmartcontract/go -l golang
```

Install in **peer0.webcash.kshrd.com.kh**
```sh
docker exec -e "CORE_PEER_LOCALMSPID=WebcashMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/webcash.kshrd.com.kh/users/Admin@webcash.kshrd.com.kh/msp" -e "CORE_PEER_ADDRESS=peer0.webcash.kshrd.com.kh:7051" cli peer chaincode install -n kshrdsmartcontract -v 1.0 -p github.com/kshrdsmartcontract/go -l golang
```

Install in **peer1.webcash.kshrd.com.kh**
```sh
docker exec -e "CORE_PEER_LOCALMSPID=WebcashMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/webcash.kshrd.com.kh/users/Admin@webcash.kshrd.com.kh/msp" -e "CORE_PEER_ADDRESS=peer1.webcash.kshrd.com.kh:7051" cli peer chaincode install -n kshrdsmartcontract -v 1.0 -p github.com/kshrdsmartcontract/go -l golang
```

In this case, if the command successed, you should see a response indicating a status of `200` and a payload of `OK`, as illustrated by the following:

```sh
2018-07-17 08:08:26.604 UTC [viperutil] getKeysRecursively -> DEBU 001 Found map[string]interface{} value for peer.BCCSP
2018-07-17 08:08:26.605 UTC [viperutil] unmarshalJSON -> DEBU 002 Unmarshal JSON: value cannot be unmarshalled: invalid character 'S' looking for beginning of value
2018-07-17 08:08:26.605 UTC [viperutil] getKeysRecursively -> DEBU 003 Found real value for peer.BCCSP.Default setting to string SW
.
.
.
2018-07-17 08:08:27.079 UTC [msp/identity] Sign -> DEBU 04e Sign: plaintext: 0AAD070A5B08031A0B08FBC7B6DA0510...DFE3DF010000FFFF0466BFCB001E0000
2018-07-17 08:08:27.079 UTC [msp/identity] Sign -> DEBU 04f Sign: digest: FC7CF33104D0214603B78CA664651B5844C232BBDB0A280F60854AEDF6B4F78C
2018-07-17 08:08:27.206 UTC [chaincodeCmd] install -> INFO 050 Installed remotely response:<status:200 payload:"OK" >
```

### Instantiate chaincode

After we installed the chaincode, we need to instantiate it. This will initialize the chaincode on the channel - the init function of chaincode will be called.

Use the following command to instantiate the chaincode from one of the peers:

```sh
docker exec -e "CORE_PEER_LOCALMSPID=CooconMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/users/Admin@coocon.kshrd.com.kh/msp" -e "CORE_PEER_ADDRESS=peer0.coocon.kshrd.com.kh:7051" cli peer chaincode instantiate -o orderer1.kshrd.com.kh:7050 -o orderer2.kshrd.com.kh:7050 -o orderer3.kshrd.com.kh:7050 -C mychannel -n kshrdsmartcontract -l golang -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('CooconMSP.member','WebcashMSP.member')"
```

This command causes the following actions to occur:
- Spawns a chaincode Docker container to house the chaincode.
- Invokes the init function of the chaincode
- Defines the endorsement policy for our channel. In this example, the ```-P "OR ('CooconMSP.member','WebcashMSP.member')"``` statement indicates that one **(OR)** of the peers (members of organizations) is enough to endorse a transaction for it to be valid.

```sh
2018-07-17 08:24:49.403 UTC [viperutil] getKeysRecursively -> DEBU 001 Found map[string]interface{} value for peer.BCCSP
.
.
.
2018-07-17 08:24:49.449 UTC [msp/identity] Sign -> DEBU 04d Sign: plaintext: 0AB9070A6708031A0C08D1CFB6DA0510...684D53500A04657363630A0476736363
2018-07-17 08:24:49.449 UTC [msp/identity] Sign -> DEBU 04e Sign: digest: 0D8D666D5648B43302975226AD7DFCD107922A34E1F24502D70FEBAD41A659C2
2018-07-17 08:24:49.460 UTC [msp/identity] Sign -> DEBU 04f Sign: plaintext: 0AB9070A6708031A0C08D1CFB6DA0510...D8682A9FF8EA57F649FEF526ECCC1E20
2018-07-17 08:24:49.460 UTC [msp/identity] Sign -> DEBU 050 Sign: digest: F15283B312FFF83C06A4D865AA5C0164A05F4B8A1F19CB3B83EC43DCB1D53A21
```


### Execute tutorial scenario
#### Query for initial state of the ledger

To quickly verify what data are in our ledger we can query the ledger for the value of a:
```sh
docker exec -e "CORE_PEER_LOCALMSPID=CooconMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/users/Admin@coocon.kshrd.com.kh/msp" -e "CORE_PEER_ADDRESS=peer0.coocon.kshrd.com.kh:7051" cli peer chaincode invoke -o orderer1.kshrd.com.kh:7050 -o orderer2.kshrd.com.kh:7050 -o orderer3.kshrd.com.kh:7050 -C mychannel -n kshrdsmartcontract -c '{"Args":["query","a"]}'
```

#### Invoke chaincode

You can invoke the chaincode by using the CLI or Fabric SDK. In this tutorial, we will use the CLI to invoke the function in the chaincode by using the following command:

```sh
docker exec -e "CORE_PEER_LOCALMSPID=CooconMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/users/Admin@coocon.kshrd.com.kh/msp" -e "CORE_PEER_ADDRESS=peer0.coocon.kshrd.com.kh:7051" cli peer chaincode invoke -o orderer1.kshrd.com.kh:7050 -o orderer2.kshrd.com.kh:7050 -o orderer3.kshrd.com.kh:7050 -C mychannel -n kshrdsmartcontract -c '{"Args":["invoke","a","b","10"]}'
```

```sh
docker exec -e "CORE_PEER_LOCALMSPID=CooconMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/users/Admin@coocon.kshrd.com.kh/msp" -e "CORE_PEER_ADDRESS=peer1.coocon.kshrd.com.kh:7051" cli peer chaincode invoke -o orderer1.kshrd.com.kh:7050 -o orderer2.kshrd.com.kh:7050 -o orderer3.kshrd.com.kh:7050 -C mychannel -n kshrdsmartcontract -c '{"Args":["invoke","a","b","10"]}'
```

```sh
docker exec -e "CORE_PEER_LOCALMSPID=WebcashMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/webcash.kshrd.com.kh/users/Admin@webcash.kshrd.com.kh/msp" -e "CORE_PEER_ADDRESS=peer0.webcash.kshrd.com.kh:7051" cli peer chaincode invoke -o orderer1.kshrd.com.kh:7050 -o orderer2.kshrd.com.kh:7050 -o orderer3.kshrd.com.kh:7050 -C mychannel -n kshrdsmartcontract -c '{"Args":["invoke","a","b","10"]}'
```

```sh
docker exec -e "CORE_PEER_LOCALMSPID=WebcashMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/webcash.kshrd.com.kh/users/Admin@webcash.kshrd.com.kh/msp" -e "CORE_PEER_ADDRESS=peer1.webcash.kshrd.com.kh:7051" cli peer chaincode invoke -o orderer1.kshrd.com.kh:7050 -o orderer2.kshrd.com.kh:7050 -o orderer3.kshrd.com.kh:7050 -C mychannel -n kshrdsmartcontract -c '{"Args":["invoke","a","b","10"]}'
```

When you run the above command, the console output will contain `result: status:200`:
```sh
2018-07-17 08:31:01.651 UTC [viperutil] getKeysRecursively -> DEBU 001 Found map[string]interface{} value for peer.BCCSP
2018-07-17 08:31:01.651 UTC [viperutil] unmarshalJSON -> DEBU 002 Unmarshal JSON: value cannot be unmarshalled: invalid character 'S' looking for beginning of value
2018-07-17 08:31:01.651 UTC [viperutil] getKeysRecursively -> DEBU 003 Found real value for peer.BCCSP.Default setting to string SW
.
.
.
2018-07-17 08:31:16.527 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> DEBU 04f ESCC invoke result: version:1 response:<status:200 > payload:"\n 5\210_]d\305_o;7\350'y\330c\341\242w\032\251\014\031\260\317\211Ec\360\210\234\353[\022\203\001\na\022;\n\022kshrdsmartcontract\022%\n\007\n\001a\022\002\010\006\n\007\n\001b\022\002\010\006\032\007\n\001a\032\00270\032\010\n\001b\032\003230\022\"\n\004lscc\022\032\n\030\n\022kshrdsmartcontract\022\002\010\003\032\003\010\310\001\"\031\022\022kshrdsmartcontract\032\0031.0" endorsement:<endorser:"\n\nWebcashMSP\022\246\006-----BEGIN CERTIFICATE-----\nMIICJTCCAcugAwIBAgIQBLu/nZG21GQAkXhoexDPDjAKBggqhkjOPQQDAjB7MQsw\nCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMNU2FuIEZy\nYW5jaXNjbzEdMBsGA1UEChMUd2ViY2FzaC5rc2hyZC5jb20ua2gxIDAeBgNVBAMT\nF2NhLndlYmNhc2gua3NocmQuY29tLmtoMB4XDTE4MDcxNzA2MjM0NloXDTI4MDcx\nNDA2MjM0NlowXzELMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAU\nBgNVBAcTDVNhbiBGcmFuY2lzY28xIzAhBgNVBAMTGnBlZXIxLndlYmNhc2gua3No\ncmQuY29tLmtoMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAElE1IvivBhNhHVpgu\nHD/z/guAiBi3zjGGsN5sNOOeOT0VINpUcjildF8kNtTzfP7g0lzr4T5/jPdMv3Cn\n5uV7zKNNMEswDgYDVR0PAQH/BAQDAgeAMAwGA1UdEwEB/wQCMAAwKwYDVR0jBCQw\nIoAgXEkqjaImV20VFUpHRjXOf+raVc6x/G/9Qc+tr0wyWvowCgYIKoZIzj0EAwID\nSAAwRQIhAJf/Ne1oE+Dz8s+8nhXbtCcwOBzi9NejBt4aMnFQ5J2PAiAnU+bIJCAR\nHEANZJt3fIaZJ1vZQakUHGBpCeWPsOxd/Q==\n-----END CERTIFICATE-----\n" signature:"0D\002 /{an\207d\375\341D\373^\225O\020e\220\216|\205I\331\033\211\274D\017\250\305P\326(\034\002 ,\374f\247iX>\246y\314\356\307\037\t\212A$\201\307H]\036q\255\206%\351\352\224z\362f" >
2018-07-17 08:31:16.527 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 050 Chaincode invoke successful. result: status:200
```

#### Query for the new values

We did some changes to the values of our assets a and b. If you recall, we instantiated a to be 100 and invoked the 3 times by transferring 10 from a to b. So we expect the queries return the value of 70 for a and 230 for b.

> Note: The following queries could be run from any peer. Remember, to run it from the context of different peers, you need to prefix the command with the respective environment variables.

Query for `a`:

```sh
docker exec -e "CORE_PEER_LOCALMSPID=WebcashMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/webcash.kshrd.com.kh/users/Admin@webcash.kshrd.com.kh/msp" -e "CORE_PEER_ADDRESS=peer1.webcash.kshrd.com.kh:7051" cli peer chaincode invoke -o orderer1.kshrd.com.kh:7050 -o orderer2.kshrd.com.kh:7050 -o orderer3.kshrd.com.kh:7050 -C mychannel -n kshrdsmartcontract -c '{"Args":["query","a"]}'
```

As shown in the following screenshot, the value of a is 70 as expected:

```sh
2018-07-17 08:36:00.112 UTC [viperutil] getKeysRecursively -> DEBU 001 Found map[string]interface{} value for peer.BCCSP
2018-07-17 08:36:00.112 UTC [viperutil] unmarshalJSON -> DEBU 002 Unmarshal JSON: value cannot be unmarshalled: invalid character 'S' looking for beginning of value
.
.
.
2018-07-17 08:36:00.182 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 050 Chaincode invoke successful. result: status:200 payload:"70"
```

And let's query for `b`:

```sh
docker exec -e "CORE_PEER_LOCALMSPID=WebcashMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/webcash.kshrd.com.kh/users/Admin@webcash.kshrd.com.kh/msp" -e "CORE_PEER_ADDRESS=peer1.webcash.kshrd.com.kh:7051" cli peer chaincode invoke -o orderer1.kshrd.com.kh:7050 -o orderer2.kshrd.com.kh:7050 -o orderer3.kshrd.com.kh:7050 -C mychannel -n kshrdsmartcontract -c '{"Args":["query","b"]}'
```

Again, the result meets our expectation - value of `b` is 230:

```sh
2018-07-17 08:37:41.836 UTC [viperutil] getKeysRecursively -> DEBU 001 Found map[string]interface{} value for peer.BCCSP
2018-07-17 08:37:41.836 UTC [viperutil] unmarshalJSON -> DEBU 002 Unmarshal JSON: value cannot be unmarshalled: invalid character 'S' looking for beginning of value
.
.
.
2018-07-17 08:37:41.911 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 050 Chaincode invoke successful. result: status:200 payload:"230"
```

## Summary
At this point, our network is fully operational and this concludes our tutorial.

Here is a summary of the tasks that we completed:
- We installed all dependencies required to run Hyperledger Fabric v1.2. on Ubuntu 16.04 LTS.
- We retreived Hyperledger Fabric v1.2 artificats and binaries.
- We used the ```cryptogen``` tool to generate certificates and keys for the local MSP.
- We used the ```configtxgen``` tool to create our channel transaction and genesis block.
- We configured the ```docker-compose.yaml``` files to describe our networks and peers.
- We used the CLI container to perform the following tasks:
    - Create the channel
    - Deploy chaincode
    - Install chaincode
    - Query Chaincode
    - Invoke Chaincode
    <!-- - Query chaincode -->
## What is Next?
## References?
1. Hyperledger Fabric V1.1 Tutorial - https://github.com/CATechnologies/blockchain-tutorials/wiki/Tutorial:-Hyperledger-Fabric-v1.1-%E2%80%93-Create-a-Development-Business-Network-on-zLinux
2. Hyperledger Fabric Sample Codes https://github.com/hyperledger/fabric-samples/blob/release-1.2/chaincode/chaincode_example02/go/chaincode_example02.go
3. Hyperledger Fabric Sample - https://github.com/hyperledger/fabric-samples
4. Hyperledger Fabric - http://hyperledger-fabric.readthedocs.io/en/release-1.2/
5. Install Docker - https://docs.docker.com/install/linux/docker-ce/ubuntu/
6. Install Docker Compose - https://docs.docker.com/compose/install/
