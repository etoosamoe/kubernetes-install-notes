haproxy01
master01
master02
master03
worker01
worker02


### prepare

disable swap
install container d
enable cri (comment disable)
containerd config default | sudo tee /etc/containerd/config.toml

install packages in master nodes
install packages on worker nodes

hold packages versions

install haproxy
show template
ap haproxy.yml -i inventory.yml

### init cluster
sudo kubeadm init \
--control-plane-endpoint "haproxy01.ru-central1.internal:6443" \
--pod-network-cidr=10.244.0.0/16 \
--upload-certs

### get kube config
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

### join masters
join masters
kubeadm join haproxy01.ru-central1.internal:6443 --token z9y7as.qhisqsc0hq5q7e7t \
--discovery-token-ca-cert-hash sha256:34905ac7045e7b2aa019f1947d10e0b11172430d58c511aec8e54c454f6be138 \
--control-plane --certificate-key fc81e0243869c9462b1bfd93e5efd0cdec16e1f6e6f855acbf7883f18766fba6

change hostname to master01 and use locally


### install network

kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
If you use custom podCIDR (not 10.244.0.0/16) you first need to download the above manifest and modify the network to match your one.

ctr ns ls
ctr -n k8s.io containers list
kubectl get pods --all-namespaces

### join workers
join workers
kubeadm join haproxy01.ru-central1.internal:6443 --token z9y7as.qhisqsc0hq5q7e7t \
--discovery-token-ca-cert-hash sha256:34905ac7045e7b2aa019f1947d10e0b11172430d58c511aec8e54c454f6be138

### install metallb
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.11/config/manifests/metallb-native.yaml
create configuration


### install ingress
https://github.com/kubernetes/ingress-nginx
https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal-clusters
download manifest
change to LoadBalancer
apply
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/baremetal/deploy.yaml

check ingress external address

create ingress
create record in /etc/hosts
curl helloworld.my.app - работает
ping helloworld.my.app - не работает