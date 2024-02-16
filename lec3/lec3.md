# ingress

## ingress controller
nginx ingresscontroller
```sh
# rke2는 기본적으로 nginx-ingressController가 설치 되어 있다 
#kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.6.4/deploy/static/provider/cloud/deploy.yaml

```
```sh
k get pod -n ingress-nginx
k get svc -n ingress-nginx

```
## ingress-backend
ingress-backend.yaml
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
  
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
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
---
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
  labels:
    app: "httpd"
spec:
  type: ClusterIP
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 80
  selector:
    app: "httpd"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
  labels:
    app: "httpd"
spec:
  replicas: 2
  selector:
    matchLabels:
      app: "httpd"
  template:
    metadata:
      labels:
        app: "httpd"
    spec:
      containers:
      - name: httpd
        image: httpd:latest
        ports:
        - name: http
          containerPort: 80
```
```sh
k apply -f ingress-backend.yaml
```
## ingress rule
ingress-rule.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ing
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /nginx
        pathType: Prefix
        backend:
          service:
            name: nginx-svc
            port:
              number: 80
      - path: /apache
        pathType: Prefix
        backend:
          service:
            name: httpd-svc
            port:
              number: 80
```
```sh
k apply -f ingress-rule.yaml
k get ingress
curl http://43.202.54.233/nginx
curl http://43.202.54.233/apache

```
## host 기반 
ingress-host-rule.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ing
spec:
  ingressClassName: nginx
  rules:
  - host: "nginx.43.202.54.233.sslip.io"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-svc
            port:
              number: 80
  - host: "apache.43.202.54.233.sslip.io"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: httpd-svc
            port:
              number: 80
```
```
k apply -f ingress-host-rule.yaml

k get ing

curl http://nginx.43.202.54.233.sslip.io/
curl http://apache.43.202.54.233.sslip.io/
```

## TLS Termination(Self-signed)

```sh
mkdir certs
cd certs
## 인증서를 생성
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=nginxsvc/O=nginxsvc"

## TLS Secret
kubectl create secret tls nginx-tls-secret --key tls.key --cert tls.crt

## default namespace에서 조회
k get secret

```

ingress-tls.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ing
  # annotations:
  #   nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
  
spec:
  ingressClassName: nginx
  tls:
  - secretName: nginx-tls-secret
  rules:
  - host: "nginx.43.202.54.233.sslip.io"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-svc
            port:
              number: 80
```
```sh
k apply -f ingress-tls.yaml
# 브라우저에서 
http://nginx.43.202.54.233.sslip.io  ## 접속안됨

# aws lightsail의 43.202.54.233(master) 의 방화벽(network) 443 포트 추가 
https://nginx.43.202.54.233.sslip.io 

```
## redirect http to https
```yaml
annotations:
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
```
