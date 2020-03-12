# Kubernetes Research Lab for Raspberry Pi clusters

Inspired by several articles, in particular this [GitHub repository](https://github.com/jduncan-rva/kube-pi-lab), I started to build my own Kubernetes cluster using 4 Raspberry PIs 4 model B with 4 GB RAM.

This project's goal is to make deployment (and re-deployment) of Kubernetes in a Raspberry Pi cluster a repeatable automated process. 

## Tested Configuration & Bill of Material

Currently my clusters looks like that:
![Kubernetes Cluster with four Raspberry PIs](./images/raspi_cluster.png)

**Bill of Material**

Quantity | Item | Total Price | Order  
--- | --- | --- | ---
4 | Raspberry PI 4 Model B, 4 GB RAM | €231,60 | [Order here](https://www.berrybase.de/raspberry-pi-co/raspberry-pi/boards/raspberry-pi-4-computer-modell-b-2gb-ram)
4 | PoE Hat for Raspberry PI 4 | €79,60| [Order here](https://www.berrybase.de/neu/power-over-ethernet-40-poe-41-hat-f-252-r-raspberry-pi-4-3b-43-rev.-1.01)
4 | Samsung MB-MC32GA/EU EVO Plus 32 GB | €32,52 | [Order here]( https://smile.amazon.de/gp/product/B06XFSZGCC/ref=ppx_yo_dt_b_asin_title_o08_s00?ie=UTF8&psc=1)
1 | Stackable case | €19,99 | [Order here](https://smile.amazon.de/gp/product/B07Z4GRQGH/ref=ppx_yo_dt_b_asin_title_o09_s01?ie=UTF8&psc=1)
1 | Netgear GS305P 5 Port Gigabit Ethernet LAN PoE Switch  | €53,90| [Order here](https://smile.amazon.de/gp/product/B01NBBA093/ref=ppx_yo_dt_b_asin_title_o09_s00?ie=UTF8&psc=1)
5 | White Ethernet cable | €11,85 | [Order here](https://smile.amazon.de/gp/product/B01AWLJNBS/ref=ppx_yo_dt_b_asin_title_o09_s00?ie=UTF8&psc=1) 


I power all PIs over the switch. If you want to stack more than 4 Raspi you need to buy a bigger switch.

I decided to install Raspbian Lite (Raspbian GNU/Linux 10 (buster)) on the Raspberries. 

## Installation

After you have burnt the Raspbian images on to the SD cards (I use [Etcher](https://www.balena.io/etcher/) for that), insert the cards and boot up your Raspi.

Before running Ansible (I used version 2.9.5) to install the cluster ssh in to each Raspi once to ensure that the fingerprints of each machine is known.

Adpat the inventory file to your needs.

The installation will use Weave as SDN. I tried Flannel as well as Calico and had problems with both. Feel free to try it yourself. You need to adapt the configuration stored in group_vars folder.


By running 

`ansible-playbook -i inventory site.yaml`

you will get a basic Kubernetes cluster with a single control node and 3 workload nodes.

**Notes regarding the installation**
* The Kubernetes configuration file will be copied to $HOME/.kube/config_raspi. Point or add this file to the environment variable KUBECONFIG in order to access your Kubernetes cluster.
In order to use the cluster you need to set your context appropriately.

    `kubectl config use-context my-cluster-name`

* The IP addresses are hardcoded in the inventory file. You might want to remove the IP addresses. I let my cluster nodes receive IP addresses via DHCP in my LAN. Afterwards I specified that the nodes should always receive the same address. You might want to consider using static IP addresses for your nodes.
  

## Accessing the Dashboard

Open a terminal on the machine from which you want to access your dashboard and run

`kubectl proxy`

Afterwards you can access your dashboard via browser on the URL

`http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login`

In order to login select the option 'Token', run the command

`kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')`

The output looks like that:
>
>Name:         admin-user-token-zqdvp
>Namespace:    kubernetes-dashboard
>Labels:       <none>
>Annotations:  kubernetes.io/service-account.name: admin-user
>              kubernetes.io/service-account.uid: db9503d8-a847-490a-b40e-48661c6b8639
>
>Type:  kubernetes.io/service-account-token
>
>Data
>====
>namespace:  20 bytes
>token:      >eyJhbGciOiJSUzI1NiIsImtpZCI6ImtLQ2NlTzhkX3ZRbFp3UnBwZXA2ZjdvbllrYndnalJZakk2ZmVDTkRmS1EifQ.>eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2U>iOiJrdWJlcm5ldGVzLWRhc2hib2AyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi>11c2VyLXRva2VuLXpxZHZwIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkb>WluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJkYjk1MDNkOC1hODQ3>LTQ5MGEtYjQwZS00ODY2MWM2Yjg2MzkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ>9YWRtaW4tdXNlciJ9.>Modf_XjT4HgYfBzdVSXKf6hUAhfRCpH2P_v4eF4JliKaVPeLeqh4PVLpMAJ-T5zcmN1ehbbFEG5aEpiBuTWN_H_QHHVcKFMLZVL>dYdRrnhy_Rcq2fMumjEMJA6Zl_O5QT168pnJx6JoRZk0aaFLXB8xVtz_xvJ1dNE0A3lxRae1uW8QGYst_roMUtBkbpWCEVKxTCj>NYO2C1BdCZJ8E8a5AreDHII6X16broRqeQAryoW6FrzenbTcelbWiTZQdUuRf6LDwZNrNClhpjB23rZ9V4qzd5aX3kzsTBRZoKl>FTXTIh4PuVO9xoizFIE6-Ydu2SXXizt-1Q9Tye4C_HWBg
>ca.crt:     1025 bytes

Copy the token into the token entry field. 

## Looking forward

I tried to install Traefik as ingress controller, but I am stuck. The post submitted to the Traefik community, you'll find [here](https://community.containo.us/t/connection-refused-while-trying-to-connect-to-my-ingress/4795)

I am planning to try a different setup using nginx as ingress controller.

Further, I already purchased 3 SSDs which I plan to plug into the worker nodes and install a cluster file system. Originally, I intended to use Rook but it seems that I would need to compile the kernel (at least for Raspbian) myself which I don't want to get into. At least, not at the moment.

