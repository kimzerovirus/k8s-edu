# docker install(없다면 )
```sh
## install-vm에서 실행 
## ubuntu user로 실행 
sudo apt update

snap version  ## 2.61.1 되어야 한다 
## 2.61.1 아니면 아래 실행하여 version ip 한다 
# sudo snap refresh
sudo snap install docker 

sudo docker ps 

## Docker 그룹 생성(snap docker install은 docker 그룹을 만들지 않는다)
sudo addgroup --system docker

## sudo 없이 Docker 명령 실행
sudo usermod -a -G docker $USER

## 재로그인후 체크 
docker ps # 안될경우 sudo reboot 
```

# ansible argocd 설치 및 App 배포 (demo-gitOps/vas)
```sh
cd ansible
ansible-playbook -i host-vm playbook.yml -t "app-deploy, app-deploy" -e "@vars.yml"
```
- https://argocd.13.125.30.181.sslip.io/ 접속한다 
- 로그인 : admin/admin1234
- vas app이 정상적으로 생성되었는지 확인
- vas app 접속 : http://vas.13.125.30.181.sslip.io/

# demo app 수정 
```sh
## install-vm에서 실행 
vi apps/demo/src/main/java/com/example/demo/controller/DemoController.java
## return "hello world VAS !!! version : 1.0.0 "; 에서 버전업 수정 한다 

## docker build 
cd apps/demo
docker build -t [docker-hub 계정]/vas:1.0.0 . 
# ex} docker build -t saturn203/vas:1.0.0 . 
docker images
docker login 
docker push [docker-hub 계정]/vas:1.0.0
# ex} docker push saturn203/vas:1.0.0  
```

## demo-gitOps image tag update 
```sh
## install-vm에서 실행 
## demo-gitOps 없는경우 아래와 같이 git clone 한다 
cd ~
git clone https://github.com/io203/demo-gitops.git 

cd demo-gitops/
vi vas/kustomization.yaml

 # newTag: 1.0.0 수정

 git add . 
 git commit -m "vas:1.0.0 수정"
 git push origin
```

## argocd vas app auto sync
- argocd 기본적으로 180초(3분) 마다 refresh가 이루어져 자동 sync 된다  





