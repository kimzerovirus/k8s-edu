# 1.  git clone k8s-edu
```bash
## master-1에서 실행
git clone https://github.com/io203/k8s-edu.git
cd k8s-edu/lec0/multi-master

```
# 2. multi master rke2



## 2.1 master01 설치
1. lightsail에서 master1~3 개 vm 생성 한다 
2. master1에 rke2-master01-install.sh 만들고 실행 한다 
```bash
## root로 로그인

export EXTERNAL_IP=15.164.227.212
#export INTERNAL_IP=172.26.14.54

vi rke2-master01-install.sh

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
```
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

K10e82e799cd896df44dd10591faa8926798d4212defde0fa99f23e2ef51e371a3d::server:f3d96ade6997d01d423bf60995bc05be


```

## 2.4 master02 설치 
```bash
## root로 로그인 

export EXTERNAL_IP=43.203.197.229
export MASTER01_INTERNAL_IP=172.26.4.26
export TOKEN=K10e82e799cd896df44dd10591faa8926798d4212defde0fa99f23e2ef51e371a3d::server:f3d96ade6997d01d423bf60995bc05be

vi rke2-master02-03-install.sh
sh rke2-master02-03-install.sh
```

## 2.5 master03 설치 
```bash
## root로 로그인 
export EXTERNAL_IP=3.36.91.29
export MASTER01_INTERNAL_IP=172.26.4.26
export TOKEN=K107d6e7eeb425a2b62c496a1a2890760e618c9d3b9a17558c4a4a00cb7d4836b95::server:a4361e349f3facdef377283e5d6edf61


vi rke2-master02-03-install.sh
sh rke2-master02-03-install.sh
```

## 2.6 worker(agent)
```
export MASTER01_INTERNAL_IP=172.26.14.54
export TOKEN=K100c35754f608a35deb11f1340dff0b2496ab9e8a76419bd627b2e8ede4dbb9968::server:354c7d406ca162ecf7c19bae41fc2115

vi rke2-agent-install.sh

sh rke2-agent-install.sh

```