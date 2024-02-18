
# 1.  git clone k8s-edu
```bash
## master-1에서 실행
## root 로 진행 
git clone https://github.com/io203/k8s-edu.git
cd k8s-edu/lec0/multi-master

```
# 2. multi master rke2

![alt text](image.png)


## 2.1 master01 설치
1. lightsail에서 master1~3 개 vm 생성 한다 
```bash
## root로 로그인

export EXTERNAL_IP=43.203.210.49
sh rke2-master01-install.sh

source ~/.bashrc

## debug
journalctl -u rke2-server -f

kubectl version
kubectl config view 
kubectl cluster-info
kubectl get nodes
kubectl get pod -A
```

## 참고: uninstall rke2
```sh
rke2-uninstall.sh
```

## 2.2 k9s 설치 
```bash
# install k9s with snap
snap install k9s 
ln -s /snap/k9s/current/bin/k9s /snap/bin/
```

## 2.3 master01 token  
master01에서 
```
cat /var/lib/rancher/rke2/server/node-token

K10856fb17a5ce7eb3c083c9fa5b18e7aed519245dd4e29a342ffbf79865c6feb3b::server:3a0459d3712e2b9d4e4d69b6c5f38d89


```

## 2.4 master02 설치 
```bash
## master-1이 어느정도 설치가 끝나고 진행 한다 
## root로 로그인 

export EXTERNAL_IP=43.203.210.49
export MASTER01_INTERNAL_IP=172.26.8.159
export TOKEN=K10856fb17a5ce7eb3c083c9fa5b18e7aed519245dd4e29a342ffbf79865c6feb3b::server:3a0459d3712e2b9d4e4d69b6c5f38d89
sh rke2-master02-03-install.sh

```

## 2.5 master03 설치 
```bash
## root로 로그인 
export EXTERNAL_IP=43.203.210.49
export MASTER01_INTERNAL_IP=172.26.8.159
export TOKEN=K10856fb17a5ce7eb3c083c9fa5b18e7aed519245dd4e29a342ffbf79865c6feb3b::server:3a0459d3712e2b9d4e4d69b6c5f38d89

sh rke2-master02-03-install.sh
```

## 2.6 worker(agent)
```sh 
export MASTER01_INTERNAL_IP=172.26.8.159
export TOKEN=K10856fb17a5ce7eb3c083c9fa5b18e7aed519245dd4e29a342ffbf79865c6feb3b::server:3a0459d3712e2b9d4e4d69b6c5f38d89
sh rke2-agent-install.sh

```