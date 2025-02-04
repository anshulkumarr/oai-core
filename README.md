# oai-Core in a Virtualized Environment
## Description
The target [Architectural diagram](https://github.com/subhrendu1987/oaisetup/blob/main/BankOnWheelsPoster.drawio.pdf) has two parts. (a) Virtualized 5G Core and (b) gNB docker[^1]. This tutorial is about how to create Virtualized 5G Core VM. We have used Intel Core i7 systems along with Ettus B210. We have used Ubuntu 20.04 and we suggest use of same OS.

[^1]: For gNB docker use [this](https://github.com/subhrendu1987/oai-gnodeb) repository.
## VM Preparation
* Install Oracle Virtual Box and extension pack. In this case we have used 6.1.38.
* Instantiate an Ubuntu 20.04 VM with 4vCPUs, 4GB RAM, and bridged network adapter. For simplicity we are calling this instance as `Core VM` and the physical system as `Core Baremetal`
## Docker installation
We have used the following [Tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04) to install docker engine. Interested readers may go through the given tutorial to avoid `sudo` while using docker along with multiple other useful things.
	```
	
	
	sudo apt update
	
	sudo apt install apt-transport-https ca-certificates curl software-properties-common
	
	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
	
	sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
	
	apt-cache policy docker-ce
	
	sudo apt install docker-ce
	
	sudo systemctl status docker
	```
### Git clone
	```
	git clone https://github.com/subhrendu1987/oai-core
	```
### Import dockers oai-core
1. Change directory to `component` and Recombine and load all dockers. The commands related to it are given [here](https://github.com/subhrendu1987/oai-core/blob/main/component/README.md) file.
2. `cd ../docker-compose`
3. Permit all shell scripts to execute i.e. `find ./ -type f -print |grep ".sh"|xargs sudo chmod -v 777`
### Routing table adjustments
1. ping test from `GNB` to `AMF` and `AMF` to `GNB`. If ping is not successfull, then try to debug
	1. Commands to be executed in Core to enable packet forwarding from GnodeB to the external network via the `Core VM`
		```
		sudo sysctl net.ipv4.ip_forward=1
		sudo iptables -P FORWARD ACCEPT
		sudo ip route add 192.168.71.194 via <GNB Baremetal IP>
		sudo ip route add 12.1.1.0/24 via 192.168.70.134 # Forward packets to Mobiles from external sources
		```
	1. Check routing tables of `GNB Docker`, `GNB Baremetal`, `Core VM`, `Core Baremetal`
	2. For ease of operations use the Google sheet [tool](https://docs.google.com/spreadsheets/d/1n9lvu_eg6hOtkR9DOmAPyxP7QXRWBrLoCCP3OOhmwyw/edit#gid=0)

## Execute with `docker compose`
Edit `docker-compose-basic-nrf.yaml` and modify the following variables:
1. Remove all occurences of `HTTP_PROXY` and `HTTPS_PROXY` if you are not in a proxy based environment
1. Go to the service descriptions of `oai-amf` and fill up the appropriate values
1. Provide same `MCC` and `MNC` in the service descriptions of `oai-spgwu`
1. Search for the comments (i.e. #) to identify the values which might require adjustments

### Start core services
	`cd docker-compose; sudo python3 core-network.py --type start-basic --scenario 1`

### Stop core services
	`sudo python3 core-network.py --type stop-basic`
### Test and debug
1. `sudo docker logs --follow oai-amf`
### Execute GnodeB
If everything is ok, then in `oai-amf` will be able to reflect the registered gnodeB name and ID.
### Sim registration
Sim registration is done in two parts. 
1. Writing of the sim card
1. Adding credentials to the `Core VM`
For issues related to sim registration, take a look at the [tutorial](https://github.com/subhrendu1987/oaisetup/tree/main/UE).
