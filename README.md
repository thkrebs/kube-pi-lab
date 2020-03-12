# Kubernetes Research Lab for Raspberry Pi clusters

Inspired by several articles, in particular this [GitHub repository](https://github.com/jduncan-rva/kube-pi-lab), I started to build by own Kubernetes cluster using 4 Raspberry PIs modell 4. 

This project's goal is to ease deployment (and re-deployment) of Kubernetes in a Raspberry Pi cluster. 

## Tested Configuration & Bill of Material

Currently my clusters looks like that:
![Kubernetes Cluster with four Raspberry PIs](http://./images/raspi_cluster.png)

**Bill of Material**

Quantity | Item | Total Price | Order  
--- | --- | --- | ---
4 | Raspberry PI 4 Model B, 4 GB RAM | €231,60 | [Order here](https://www.berrybase.de/raspberry-pi-co/raspberry-pi/boards/raspberry-pi-4-computer-modell-b-2gb-ram)
4 | PoE Hat for Raspberry PI 4 | €79,60| [Order here](https://www.berrybase.de/neu/power-over-ethernet-40-poe-41-hat-f-252-r-raspberry-pi-4-3b-43-rev.-1.01)
4 | Samsung MB-MC32GA/EU EVO Plus 32 GB | €32,52 | [Order here]( https://smile.amazon.de/gp/product/B06XFSZGCC/ref=ppx_yo_dt_b_asin_title_o08_s00?ie=UTF8&psc=1)
1 | Stackable case | €19,99 | [Order here](https://smile.amazon.de/gp/product/B07Z4GRQGH/ref=ppx_yo_dt_b_asin_title_o09_s01?ie=UTF8&psc=1)
1 | Netgear GS305P 5 Port Gigabit Ethernet LAN PoE Switch  | €53,90| [Order here](https://smile.amazon.de/gp/product/B01NBBA093/ref=ppx_yo_dt_b_asin_title_o09_s00?ie=UTF8&psc=1)
5 | White Ethernet cable | €11,85 | [Order here](https://smile.amazon.de/gp/product/B01AWLJNBS/ref=ppx_yo_dt_b_asin_title_o09_s00?ie=UTF8&psc=1) 


So I power all PIs over the switch. If you want to stack more than 4 Raspi you need to buy a bigger switch.

I decided to install Raspbian Lite (Raspbian GNU/Linux 10 (buster)) on the Raspberries. 

## Installation

After you have burnt the Raspbian images on to the SD cards (I use [Etcher](https://www.balena.io/etcher/) for that), insert the cards and boot up your Raspbi.

Before running Ansible (I used version 2.9.5) to install the cluster ssh in to each Raspi once to ensure that the fingerprints of each machine is known.

Adpat the inventory file to your needs.

The installation will use Weave as SDN. I tried Flannel as well as Calico and had problems with both. Feel free to try it yourself. You need to adapt the configuration stored in group_vars folder.


By running 

`ansible-playbook -i inventory site.yaml`

you will get a basic Kubernetes cluster with a single control node and 3 workload nodes.


## Looking forward

I tried to install Traefik as ingress controller, but I am stuck. The post submitted to the Traefik community, you'll find [here](https://community.containo.us/t/connection-refused-while-trying-to-connect-to-my-ingress/4795)

I am planning to try a different setup nginx as ingress controller.

Further, I already purchased a 3 SSDs which I plan to plug into the worker nodes and install a cluster file system. Originally, I intended to use Rook but it seems that I would need to compile the kernel (at least for Raspbian) myself which I don't want to get into. At least, not at the moment.

