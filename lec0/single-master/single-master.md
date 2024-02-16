# 1.  git clone k8s-edu
```bash
## master-1에서 실행
## root로 실행
sudo su -
git clone https://github.com/io203/k8s-edu.git
cd k8s-edu/lec0/single-master
```

# 2. single-master rke2

## 2.1 master 설치
```bash

## root로 실행  
export EXTERNAL_IP=3.36.65.188
sh rke2-single-master-install.sh

source ~/.bashrc

watch kubectl get pod -A

## ubuntu유저 kubeconfig 설정
## ubuntu유저로 전환후 실행
exit

git clone https://github.com/io203/k8s-edu.git
cd k8s-edu/lec0/single-master

sh master-ubuntu-user-kubeconfig.sh

```

## 2.2 master token  

master vm에서 실행  
```sh
sudo cat /var/lib/rancher/rke2/server/node-token

K10a8234470479b3708bf1dd224d8cf8ce5c909318f0735b270b66c678e8d5fd565::server:a83750658fed20e728a8b097897e7633
```

## 2.3 agent 설치
```sh
## worker-1/worker-2 실행 
## root로  설치 한다 
sudo su -
git clone https://github.com/io203/k8s-edu.git
cd k8s-edu/lec0/single-master

## master의 private IP를 입력 해야 함 
export MASTER01_INTERNAL_IP=172.26.5.131
export TOKEN=K10a8234470479b3708bf1dd224d8cf8ce5c909318f0735b270b66c678e8d5fd565::server:a83750658fed20e728a8b097897e7633
sh rke2-single-agent-install.sh
```

