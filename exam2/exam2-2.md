# DaemonSet

fluent-bit-daemonset.yaml
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentbit
spec:
  selector:
    matchLabels:
      name: fluentbit
  template:
    metadata:
      labels:
        name: fluentbit
    spec:
      containers:
      - name: aws-for-fluent-bit
        image: amazon/aws-for-fluent-bit:2.1.0
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true       
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
      
```
```sh
k apply -f fluent-bit-daemonset.yaml
k get ds 

```

# StatefulSet
nginx-statefulset.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless-svc
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-statefulset
spec:
  selector:
    matchLabels:
      app: nginx 
  serviceName: "nginx-headless-svc"
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx 
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
```
- 데모를 위해서 Volume 없이 생성한 예이다 
- StatefulSet은 serviceName을 요구하며 해당 서비스는 headless 이어야 한다 

```sh
k apply -f nginx-statefulset.yaml
k get sts

# api및 단축키 조회
k api-resources
```

## headless service 
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless-svc
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx

```
- headless 서비스는 `clusterIP: None` 으로 설정 하면 된다 

```
k get svc 

NAME                 TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
kubernetes           ClusterIP      10.43.0.1     <none>        443/TCP        26h
nginx-headless-svc   ClusterIP      None          <none>        80/TCP         12m

k describe svc nginx-headless-svc
```
- cluster-ip에 ip가 할당 되지 않는다  
- 따라서 다음과 같이 호출 해야 한다 
```sh

## mycurlpod에서 실행 
curl nginx-headless-svc.default.svc.cluster.local

nslookup nginx-headless-svc.default.svc.cluster.local
-----
Server:         10.43.0.10
Address:        10.43.0.10:53

Name:   nginx-headless-svc.default.svc.cluster.local
Address: 10.42.1.52
Name:   nginx-headless-svc.default.svc.cluster.local
Address: 10.42.1.51
Name:   nginx-headless-svc.default.svc.cluster.local
Address: 10.42.1.53

```