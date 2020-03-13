# Adding NFS as storage provider

In order to utilize my NAS at home, I added NFS support in my Kubernetes cluster. The article on [Medium](https://medium.com/@myte/kubernetes-nfs-and-dynamic-nfs-provisioning-97e2afb8b4a9) following article.
Finally, I installed the dynamic provisioning approach where a persistence volume claum will automatically trigger the creation of a persistence volume.

## Installation steps

* Install Helm. I used version 3.1.1
* Add the official Helm repository
  
    `helm repo add stable https://kubernetes-charts.storage.googleapis.com/`

* Create a volume on a NFS server and make it accessible in your network
* Install the NFS provider by running
  
    `helm install nfs-client-provisioner --set nfs.server=192.168.178.80 --set nfs.path=/volume1/clustervol  --set image.repository=quay.io/external_storage/nfs-client-provisioner-arm stable/nfs-client-provisioner`

    insert the IP address of your NFS server in the option `nfs.server=xxx.xxx.xxx.xx` and your volume path in `nfs.path=xxxxxxx`

## Test 

If everyting went right you can test the setup by applying the YAML files in the directory `nfs-dynamic` . 

> kubectl apply -f test_claim.yml
> kubectl apply -f test_pod.yml

The first YAML file creates a persistent volume claim. The second YAML file spawns up a pod which writes the file SUCCESS into the NFS volume and terminates afterwards.

In order to verify the result look for the file SUCCESS in your NFS volume.
