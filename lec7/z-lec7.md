# lecture-7
- install-vm에서 실행 
- ubuntu유저로  실행   
```sh
# cd ~
# git clone https://github.com/io203/k8s-edu.git
cd  ~/k8s-edu/lec7
```


# 1. worker node 3개로 구성한다 
- demo를 위해서 worker node를 3개로 구성한다 

# 2. node lables
```sh
kubectl get nodes
kubectl get nodes --show-labels 
kubectl get nodes --show-labels | grep kubernetes.io/hostname


## node에 label 설정하기 
kubectl get nodes
kubectl label nodes ip-172-26-12-185 nodename=worker-1 web=true ## worker-1
kubectl label nodes ip-172-26-0-29 nodename=worker-2 web=true  ## worker-2
kubectl label nodes ip-172-26-11-60 nodename=worker-3 db=true ## worker-3
kubectl get nodes --show-labels | grep nodename
kubectl get nodes --show-labels | grep db

## label 삭제 
kubectl label nodes ip-172-26-11-60  db-
kubectl get nodes --show-labels | grep db

```
# 3. nodeSelector 적용 nginx 배포
```sh
## web: "true"
kubectl apply -f nodeSelector.yaml

```

# 4. nodeAffinity
```sh
## 기존 예제 삭제
kubectl delete -f  nodeSelector.yaml

## requiredDuringSchedulingIgnoredDuringExecution
## web: "true"
kubectl apply -f  nodeRequireAffinity.yaml
## 삭제
kubectl delete -f nodeRequireAffinity.yaml

## preferredDuringSchedulingIgnoredDuringExecution
## 맞지 않는 web=true1 로 설정해서 생성하기 
kubectl apply -f  nodePreferAffinity.yaml
## 삭제 
kubectl delete -f  nodePreferAffinity.yaml
```

# 5. podAffinity
```sh 
kubectl apply -f nginx-deploy.yaml
kubectl get pod 
kubectl label pods nginx-6cf7cbb4d6-bmpvc security=S1
kubectl label pods nginx-6cf7cbb4d6-gng4w security=S2
kubectl label pods nginx-6cf7cbb4d6-vppbw security=S3

kubectl apply -f podAffinity.yaml   ## security=S2
## 확장해도 같은 pod의 위치에 배치됨
kubectl scale --current-replicas=4 --replicas=5 deployment/nginx
## 삭제
kubectl delete -f podAffinity.yaml

```

# 6. podAntiAffinity
```sh
## replicas=3
kubectl apply -f podAntiAffinity.yaml
## 1개는 pending 이 된다 

## replicas=2 줄여본다 
kubectl scale --current-replicas=3 --replicas=2 deployment/nginx

## update nginx version 
## nginx:1.17 --> nginx:1.18
kubectl set image deployment/nginx nginx=nginx:1.18
## pending 발생 

## pending 발생하여 다음과 같이   strategy.rollingUpdate의 maxUnavailable 으로 제어
## replicas=2, nginx:1.18
kubectl apply -f podAntiAffinity2.yaml
kubectl set image deployment/nginx nginx=nginx:1.19

## 다른 방법으로  strategy.rollingUpdate의 maxSurge 로 제어 
## replicas=2, nginx:1.21
kubectl apply -f podAntiAffinity3.yaml
kubectl set image deployment/nginx nginx=nginx:1.22

## clear 
kubectl delete -f podAntiAffinity3.yaml

```
# 7. Taint/Toleration
```sh
## node에 taint 설정 
kubectl taint nodes ip-172-26-12-185 oss=monitoring:NoSchedule ## worker-1
## taint 확인
kubectl describe node ip-172-26-12-185 

## nginx deploy web=true 한다
kubectl apply -f nginx-deploy.yaml
## nginx pod가 생성된 node를 확인한다 (모두  worker-2 생성)
## taint 삭제 
kubectl taint nodes ip-172-26-12-185 oss-
## nginx 1개의 pod를 삭제하여 다른 노드에 설치 되는지 확인 한다 (worker-1에도 생성 되는지 확인)

## nginx 를 모두 삭제 한다 
kubectl delete -f nginx-deploy.yaml

## 다시 label web=true인 노드의 1개에 taint를 설정한다 (worker-2)
kubectl taint nodes ip-172-26-0-29  oss=monitoring:NoSchedule
kubectl describe node ip-172-26-0-29

## Toleration이 설정된 nginx를 배포한다 
kubectl apply -f nginx-deploy-toleration.yaml

## clear 
kubectl delete -f nginx-deploy-toleration.yaml

```

