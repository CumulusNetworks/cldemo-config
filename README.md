Demo Routing Configurations
===========================
This demo and these configurations are written to be used with the [cldemo-vagrant](https://github.com/cumulusnetworks/cldemo-vagrant) reference topology. Before running this demo, follow the Prerequisites and Getting Started instructions for your system (see below).

The configuration files in this repository will be placed on the appropriate devices in order to set up the desired Routing Protocol between the leafs and spines, and will configure a Layer 2 bridge on each leaf top-of-rack switch for the servers in that rack.

This Github repository contains the configuration files necessary for setting up Layer 3 routing on a [CLOS topology](http://www.networkworld.com/article/2226122/cisco-subnet/clos-networks--what-s-old-is-new-again.html) using Cumulus Linux and FRRouting.

Ansible is used to quickly deploy configuration from the OOB server to devices on the network.

Topology
--------
This demo runs on a spine-leaf topology with two single-attached hosts. 
The helper out-of-band management network  provides access to eth0 on all of the in-band devices.

         +------------+       +------------+
         | spine01    |       | spine02    |
         |            |       |            |
         +------------+       +------------+
         swp1 |    swp2 \   / swp1    | swp2
              |           X           |
        swp51 |   swp52 /   \ swp51   | swp52
         +------------+       +------------+
         | leaf01     |       | leaf02     |
         |            |       |            |
         +------------+       +------------+
         swp1 |                       | swp2
              |                       |
         eth1 |                       | eth2
         +------------+       +------------+
         | server01   |       | server02   |
         |            |       |            |
         +------------+       +------------+



Deploying An Example Configuration
------------------------

Eight different configuration examples are included:

 * OSPF Numbered
 * OSPF Unnumbered
 * BGP Numbered
 * BGP Unnumbered
 * OSPF Numbered with IPv6
 * OSPF Unnumbered with IPv6
 * BGP Numbered with IPv6
 * BGP Unnumbered with IPv6

### 1). Install Prerequisites

Go to the [prequisites list](https://github.com/CumulusNetworks/cldemo-vagrant#prerequisites-and-getting-started) associated with the reference topology to download an install the required software.

### 2). Clone or Download the Reference Topology

To Clone the software (git must be installed already on your machine):

    git clone https://github.com/cumulusnetworks/cldemo-vagrant

If git is not installed on your machine and you'd rather [download the code directly](https://github.com/CumulusNetworks/cldemo-vagrant/archive/master.zip) make sure to unzip the file once it has been downloaded.

### 3). Start the VMs
After obtaining the software, move in to the directory containing the software.

    cd cldemo-vagrant

Use the "vagrant up" command as shown below to start the different VMs that will be used in this demo.

    vagrant up oob-mgmt-server oob-mgmt-switch leaf01 leaf02 spine01 spine02 server01 server02

### 4). Login to the Management Server
Use SSH to login to the out-of-band management server.

    vagrant ssh oob-mgmt-server

### 5). Download the Routing Configurations

    git clone https://github.com/cumulusnetworks/cldemo-config-routing
    cd cldemo-config-routing

### 6). Push Configuration Files to Devices

ansible-playbook bgp-unnumbered.yml

Note the keyword "bgp-unnumbered" in the command; this can be replaced with whatever example configuration you would like to deploy (such as "ospf-unnumbered" or "ospf-numbered" ).

 * OSPF Numbered --> "ospf-numbered"
 * OSPF Unnumbered --> "ospf-unnumbered"
 * BGP Numbered --> "bgp-numbered"
 * BGP Unnumbered --> "bgp-unnumbered"
 * OSPF Numbered with IPv6 --> "ospf-numbered-ipv6"
 * OSPF Unnumbered with IPv6 --> "ospf-unnumbered-ipv6"
 * BGP Numbered with IPv6 --> "bgp-numbered-ipv6"
 * BGP Unnumbered with IPv6 --> "bgp-unnumbered-ipv6"


### 7). Experiment
Login to server01 and ping server02.

    ssh server01
    ping 172.16.2.101


Verifying Routing
-----------------
Running the demo is easiest with two terminal windows open. One window will log into server01 and ping server02's IP address. The second window will be used to deploy new configuration on the switches.

*In terminal 1*

    vagrant ssh oob-mgmt-server
    sudo su - cumulus
    cd cldemo-config-routing
    ansible-playbook deploy-bgp-unnumbered.yml

*In terminal 2*

    vagrant ssh oob-mgmt-server
    sudo su - cumulus
    ssh server01
    ping 172.16.2.101

*In terminal 1*

    ansible-playbook deploy-bgp-unnumbered.yml -l network
    # wait and watch connectivity drop and then come back
    ansible-playbook deploy-bgp-numbered.yml -l network
    # again
    ansible-playbook deploy-bgp-numbered-ipv6.yml
    # this will reboot server01, so you'll need to log back in in terminal 2


Verify High Availability
------------------------
Using a routing protocol such as BGP or OSPF means that as long as one spine is still running, the network will automatically learn a new route and keep the fabric connected. This means that you can do rolling upgrades one spine at a time without incurring any downtime.

*In terminal 1*

    vagrant ssh oob-mgmt-server
    sudo su - cumulus
    cd cldemo-config-routing
    ansible-playbook deploy-bgp-numbered.yml

*In terminal 2*

    vagrant ssh oob-mgmt-server
    sudo su - cumulus
    ssh server01
    ping 172.16.2.101

*In terminal 3*

    vagrant destroy -f spine01
    # note that the pings may hiccup a bit, but will keep going
    vagrant destroy -f spine02
    # now pings will totally fail
    vagrant up spine01 spine02

*In terminal 1*

    ansible-playbook deploy-bgp-numbered.yml -l spine
    # watch Terminal 2, and pings will return


Using Quagga Dry Runs for Syntax Checking
-----------------------------------------
You can use Quagga's dry run functionality to check the syntax of Quagga configuration without applying the changes.

    vtysh -f Quagga.conf --dryrun

The syntax of all of the Quagga.conf files in this repository can be verified using the following line in bash.

     for i in `find  | grep Quagga.conf`; do vtysh -f $i --dryrun; done ;
