

# pod 생성

## kubectl run
```bash
kubectl run my-nginx --image=nginx

kubectl delete pod/my-nginx
```


## pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest
```

```bash
kubectl create -f pod.yaml
kubectl apply -f pod.yaml

kubectl delete -f pod.yaml

```

## pod 조회 
```bash
k get pod 
k get pod -o wide
k describe pod my-nginx 
k events pod/my-nginx 
k logs pod/my-nginx 
k logs -f pod/my-nginx 
k exec -it my-nginx-75454 -- /bin/bash
curl http://10.42.1.15


```

## Replication Controller
rc.yaml

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
          - containerPort: 80

```
```bash
k create -f rc.yaml

k delete -f rc.yaml
```


## ReplicaSet

rc.yaml

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-nginx
  labels:
    app: nginx
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
          - containerPort: 80

```
```bash
k create -f rs.yaml

k delete -f rs.yaml
```
- 레플리카셋을 직접 사용하기보다는 디플로이먼트를 사용하는 것을 권장
- kubectl get rs

replicaset은 replication controller와 똑같이 동작하지만 더 풍부한 표현식 pod selector를 갖는다.
```yaml
apiVersion: apps/v1
kind: ReplicaSet                                    
apiVersion: apps/v1                            
metadata:
  name: myrs
spec:
  replicas: 2  
  selector:                  
    matchExpressions:                             
      - {key: myname, operator: In, values: [niraj, krishna, somesh]}
      - {key: env, operator: NotIn, values: [production]}
  template:      
    metadata:
      name: testpod7
      labels:              
        myname: Bhupinder
    spec:
     containers:
       - name: c00
         image: ubuntu
         command: ["/bin/bash", "-c", "while true; do echo Technical-Guftgu; sleep 5 ; done"]
```

## deployment
- 파드와 레플리카셋(ReplicaSet)에 대한 선언적 업데이트를 제공한다
- Rolling Update
nginx-deploy.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3 
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2 
        ports:
        - containerPort: 80
```
### 조회
```sh
kubectl apply -f deploy.yaml
kubectl describe deployment nginx-deployment
kubectl get pods -l app=nginx
kubectl get pods --show-labels
```
### update 


vi deploy.yaml 
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3 
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15   ## Update the version of nginx from 1.14 to 1.15
        ports:
        - containerPort: 80
```
```
k apply -f deploy.yaml
k describe deployment nginx-deployment  
k get rs ## replicas 정보확인( DESIRED, CURRENT)
```

## deployment 삭제 
```bash
kubectl delete deployment nginx-deployment
kubectl delete -f deploy.yaml

```

## rollout 
- kubernetes 클러스터에서 롤아웃 작업을 관리하는 명령어이며  이 명령어를 사용하면, 디플로이먼트, 데몬셋, 상태풀셋 등의 리소스에 대한 롤아웃 작업을 수행할 수 있다
- rollout를 하기 위해서는 --record 옵션을 둘수 있다 하지만 이것은 deprecated 되어서 앞으로 사용하지 말것
- rollout 명령어 
  - kubectl rollout status: 롤아웃 작업의 상태를 확인
  - kubectl rollout history: 롤아웃 작업의 이력을 확인
  - kubectl rollout undo: 롤아웃 작업을 취소하고 이전 버전으로 롤백
  - kubectl rollout restart: 롤아웃 작업을 재시작 
  - kubectl rollout pause/resume: 롤링 업데이트를 일시 중지하거나 다시 시작

```bash
k apply -f deploy.yaml
k get deploy 
# k rollout status deploy/nginx-deployment --record
k rollout status deploy/nginx-deployment

k rollout history deploy/nginx-deployment
----
REVISION  CHANGE-CAUSE
1         <none>

```

### rollout history CHANGE-CAUSE 설정 

deploy-rollout.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  annotations:
    kubernetes.io/change-cause: "image updated to 1.14"

spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3 
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14
        ports:
        - containerPort: 80

```
```
k rollout history deploy/nginx-deployment
REVISION  CHANGE-CAUSE
1         image updated to 1.14
```
### image update
```sh
# nginx:1.15
kubectl set image deployment/nginx-deployment nginx=nginx:1.15
kubectl annotate deployment/nginx-deployment kubernetes.io/change-cause="image updated to 1.15"

# nginx:1.16
kubectl set image deployment/nginx-deployment nginx=nginx:1.16
kubectl annotate deployment/nginx-deployment kubernetes.io/change-cause="image updated to 1.16"

k rollout history deploy/nginx-deployment

```

### rollout undo 
```sh
k rollout undo deploy/nginx-deployment
k describe deployment nginx-deployment 
k rollout history deploy/nginx-deployment

# 지정된 revision으로 undo
k rollout undo deploy/nginx-deployment --to-revision=1
```




