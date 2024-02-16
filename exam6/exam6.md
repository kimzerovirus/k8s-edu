# CI/CD 
- git : github
- image Registry: docker-hub
- giOps: github
- build :  docker build
- deploy: argocd
- github/docker-hub 계정 필요 

# install argo-cd 
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

## argocd password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
## rke2 Updating Nginx Helm
- rke2에 미리 설치된 rke2-ingress-nginx-controller는 enable-ssl-passthrough 에 대한 설정이 없다 
- 다음과 같이 설정을 추가 할수 있다 
```bash
kubectl apply -f exam6/rke2-ingress-nginx.yaml
```
- 1개씩 update 가 된다 

## lightsail master 서버 443 오픈 
- master-1 서버의 network에서 443 추가 
  
argocd-ingress
```sh
kubectl apply -f exam6/argocd-ing.yaml
```
### access argocd ui
- https://argocd.15.165.75.251.sslip.io/
- admin/aX5yeDkjOOUdBel8
- changepassword: admin1234

## docker 설치 
```bash
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
## docker build
- exam6 > apps > demo
- demo 프로젝트에서 docker build를 한다 
- -t 의 saturn203 은 각자 개인 docker-hub계정으로 변경한다 
```sh
docker info
docker login
docker build -t saturn203/vas:0.0.1 .   ## 마지막의 .을 생략하면 안됨
docker images
docker run -p 8080:8080 saturn203/vas:0.0.1 
curl localhost:8080 
docker push saturn203/vas:0.0.1

```
- docker-hub에서 push 사항을  확인한다 

## demo-gitops 
- 각자 개인 github 에서 demo-gitops 를 public으로 생성한다 
- 생성된 demo-gitops 를 클론 한다 
```sh
cd ~
git clone [repository-uri]
## git clone https://github.com/io203/demo-gitops.git 
```
- vas-gitops 안에 exam6 > vas 폴더를 copy 한다 
```
 cp -rf ~/k8s-edu/exam6/vas ~/demo-gitops/
 cd ~/demo-gitops
 git push origin main
```

## argocd git repository 설정 
- argocd 홈  >  Settings > Repositories > connect REPO
- VIA HTTPS 선택 
- type: git
- project: default
- Repository URL : https://github.com/io203/demo-gitops.git
- Username :  각 개인 계정 
- Password :  개인 github >  Settings > Developer Settings > Personal access tokens > Tokens(classic) >  Generate new token >  token값

## vas namespace 생성 
```
kubectl create ns vas 
```

## argocd applications
- Applications > NEW APP
- Name: vas
- Project Name: default
- Repository URL :  선택 
- Revision: main
- Path :  vas 선택 
- Cluster URL :  https://kubernetes.default.svc 선택 
- Namespace:  vas
- kustomize : Images 부분에 이미지와 tag 버전이 맞는지 확인 
- 위의 CREATE 버튼 클릭
- SYNC 버튼 클릭 >  SYNCRONIZE

## vas 서비스 확인 
- http://vas.15.165.75.251.sslip.io/

## clear
```
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
