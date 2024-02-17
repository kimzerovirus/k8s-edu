# lecture-1
- install-vm에서 실행 
- ubuntu유저로  실행   
```sh
cd ~
git clone https://github.com/io203/k8s-edu.git
cd  k8s-edu/lec2
```


# Service 

## nginx deployment 배포 
```bash
## 이전 배포 clear
kubectl delete -f deploy.yaml

## 서비스용 배포 
kubectl apply -f deploy.yaml

```
## clusterIP 
clusterip-svc.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP

```
```sh
k apply -f clusterip-svc.yaml

k get svc

curl http://10.43.49.26

k  exec -it nginx-deployment-7f5b678dcb-49lhg  -- bash

echo nginx-1 > /usr/share/nginx/html/index.html
echo nginx-2 > /usr/share/nginx/html/index.html
echo nginx-3 > /usr/share/nginx/html/index.html

curl http://10.43.49.26

# endpoints
k get endpoints 
k get ep

```

## nodePort
nodeport-svc.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      nodePort: 30001
      port: 80
      targetPort: 80
  type: NodePort

```
```bash

k apply -f nodeport-svc.yaml

k get svc
# cluster 내부에서 
k get pod -o wide
k get nodes -o wide
# 내부에서 pod의 worker 노드에서 
curl 172.26.11.143:30001
# 내부에서 pod의 master 노드에서 
curl 172.26.6.180:30001

# 클러스 외부의 Browser에서 
# aws worker node network   방화벽 30001 번 오픈 
# worker node external-ip 로 접속 
http://43.202.53.29:30001
```

## LoadBalancer
loabalancer-svc.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```
```
k apply -f loabalancer-svc.yaml
## pending 확인
```

## externalName
external-svc.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: externalname1
spec:
  type: ExternalName
  externalName: naver.com
```
```sh
k apply -f external-svc.yaml

# curl 이미지로 pod 생성 
kubectl run mycurlpod --image=curlimages/curl -i --tty -- sh

curl -L externalname1.default.svc.cluster.local
curl -L externalname1.default.svc.cluster.local
```