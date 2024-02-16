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
# demo app 수정 
```sh
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

# ansible argocd 설치 및 App 배포 
```sh
cd ansible
ansible-playbook -i host-vm playbook.yml -t "argocd" 
ansible-playbook -i host-vm playbook.yml -t "arogocd-pwd" -e "@vars.yml"
```


## demo-gitOps clone (없는경우)
```
cd ~
git clone https://github.com/io203/demo-gitops.git 

cd demo-gitops/
vi vas/

```
