# 1.  git clone k8s-edu
```bash
## master-1에서 실행
git clone https://github.com/io203/k8s-edu.git
cd k8s-edu/lec0/single-master
```

# 2. single-master rke2

## 2.1 master 설치
```bash

## ubuntu 로그인 

export EXTERNAL_IP=3.38.244.98
sudo sh rke2-stand-master-install.sh

source ~/.bashrc

```
~/.bashrc
## 2.2 master token  

master vm에서 실행  
```sh
cat /var/lib/rancher/rke2/server/node-token

K10b4f22006f5b7a513a3a6f1cf08fdae8d393403078b5fc18ca74ea01b0d77b7c0::server:140d412dcc5a2d60ae82a9ca23604db4
```

## 2.3 agent 설치
```sh
## master의 private IP를 입력 해야 함 
export MASTER01_INTERNAL_IP=172.26.12.109
export TOKEN=K10b4f22006f5b7a513a3a6f1cf08fdae8d393403078b5fc18ca74ea01b0d77b7c0::server:140d412dcc5a2d60ae82a9ca23604db4
sh rke2-stand-agent-install.sh
```

