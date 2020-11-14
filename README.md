# Armbian cluster

This is a setup tool used to run a small Armbian cluster

## Setup

* Setup the SD card with Armbian/Buster. Download armbian and use the `burn` script
* Setup Armbian/Buster networking: run the `setconfig` script, check the instructions
* Boot. Find the DHCP lease and connect via ssh. Armbian default root password is one, two, three, four. The first connection will force to modify it.
* Follow the instructions attentively. Do not modify locale/language.
```
$ ssh root@192.168.1.26
...
root@192.168.1.26's password: 1234
...
New root password: ************
Repeat password: ************
...
Do you want to set locales and console keyboard automatically from your location [Y/n] N
...
Please provide a username (eg. your forename): docker
Create password: ************
Repeat password: ************
...
Please provide your real name (eg. John Doe): Docker
...
```
* Change the password!
* Modify the hostname: `vi -o /etc/hosts /etc/hostname`
* Move the /tmp/interfaces file to /etc/network/interfaces. Set the missing parameters.
* Run the following commands to disable NetworkManager and update:
```
rm -v /etc/systemd/system/dbus-org.freedesktop.nm-dispatcher.service
rm -v /etc/systemd/system/multi-user.target.wants/NetworkManager.service
apt update
apt dist-upgrade
```
* Reboot.
# https://wiki.learnlinux.tv/index.php/How_to_build_your_own_Raspberry_Pi_Kubernetes_Cluster
# https://www.youtube.com/watch?v=B2wAJ5FLOYw

sudo curl -sL get.docker.com|sh
sudo usermod -aG docker rodolfoap
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

sed -i '/net.ipv4.ip_forward/s/#//' /etc/sysctl.conf
cat > /etc/docker/daemon.json << EOF
{
   "exec-opts": ["native.cgroupdriver=systemd"],
   "log-driver": "json-file",
   "log-opts": {
     "max-size": "100m"
   },
   "storage-driver": "overlay2"
}
EOF
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubeadm
update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo apt-get install -y kubectl
sudo apt-get install -y kubelet
lsblk
wget https://raw.githubusercontent.com/docker/docker/master/contrib/check-config.sh -O cgroups_check && chmod +x cgroups_check
./cgroups_check
swapoff -a
On the master:
	kubeadm init --pod-network-cidr 10.10.0.0/16 --service-cidr 10.11.0.0/16

Which will output something like...

	Your Kubernetes control-plane has initialized successfully!

	To start using your cluster, you need to run the following as a regular user:

		mkdir -p $HOME/.kube
		sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
		sudo chown $(id -u):$(id -g) $HOME/.kube/config

	You should now deploy a pod network to the cluster.
	Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
		https://kubernetes.io/docs/concepts/cluster-administration/addons/

	Then you can join any number of worker nodes by running the following on each as root:

	kubeadm join 192.168.1.91:6443 --token o255i5.bian2b1m6hcd3yvn --discovery-token-ca-cert-hash sha256:134e4bef7cee2b5548e5ceb04bbf3c5a0a7ca3b7dda66f9e14588a685141edbc

AS A REGULAR USER:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
export KUBECONFIG=$HOME/.kube/config
echo "export KUBECONFIG=$HOME/.kube/config" | tee -a ~/.bashrc

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl get nodes,all,pv,pvc,ep
kubectl get pods --all-namespaces
sudo journalctl -u service-name.service

# Make the master a worker
kubectl taint nodes --all node-role.kubernetes.io/master-

ON EACH NODE, as root:

kubeadm join 192.168.1.91:6443 --token o255i5.bian2b1m6hcd3yvn --discovery-token-ca-cert-hash sha256:134e4bef7cee2b5548e5ceb04bbf3c5a0a7ca3b7dda66f9e14588a685141edbc

