# -*- mode: ruby -*-
# vi: set ft=ruby :

# This script to install Kubernetes will get executed after we have provisioned the box 
$script = <<-SCRIPT
echo install kubernetes
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl

echo kubelet requires swap off
swapoff -a

echo keep swap off after reboot
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

echo update cgroups
sed -i '0,/ExecStart=/s//Environment="KUBELET_EXTRA_ARGS=--cgroup-driver=cgroupfs"\n&/' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

# echo initialize kubernetes
cat << 'EOF' > /tmp/config.yaml
nodeName: vagrant
apiServerCertSANs:
- 192.168.66.100
apiServerExtraArgs:
  enable-admission-plugins: MutatingAdmissionWebhook,ValidatingAdmissionWebhook,Initializers
  runtime-config: admissionregistration.k8s.io/v1alpha1
EOF
kubeadm init --config /tmp/config.yaml

echo Copying credentials to /home/vagrant...
sudo --user=vagrant mkdir -p /home/vagrant/.kube
cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config
SCRIPT

$user_script = <<-SCRIPT
kubectl apply -f https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')
kubectl taint nodes --all node-role.kubernetes.io/master-
echo "Copy the creds from /etc/kubernetes/admin.conf to your host machine to use kubectl from there"
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-16.04"  
  config.vm.provision "docker"

  config.vm.network "private_network", ip: "192.168.66.100"
  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 4
  end
  
  config.vm.provision "shell", inline: $script
  config.vm.provision "shell", inline: $user_script, privileged: false
end
