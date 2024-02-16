# namespace

## web1 namespace
```sh
k create ns web1
```
nginx.yaml
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
```
```sh
k apply -f nginx.yaml -n web1
```

## web2 namespace
```sh
k create ns web2

```
httpd.yaml
```yaml
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
k apply -f httpd.yaml -n web2
```
## ingress 
web1-ing.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web1-ing
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
```
```sh
k apply -f web1-ing.yaml -n web1

curl http://nginx.43.202.54.233.sslip.io
```

web2-ing.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web2-ing
spec:
  ingressClassName: nginx
  rules:
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
```sh
k apply -f web2-ing.yaml -n web2

curl http://apache.43.202.54.233.sslip.io
```

# ConfigMap
configmap-example.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-configmap
data:
  index.html: |
    <html><body><h1> ===== nginx configmap test index html ==== </h1></body></html>

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

        volumeMounts:
          - name: nginx-index-config-vol
            mountPath:  /usr/share/nginx/html/index.html
            subPath: index.html      
      
      volumes:
      - name: nginx-index-config-vol
        configMap:
          name: nginx-configmap
          items:
            - key: index.html
              path: index.html
---
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
k apply -f configmap-example.yaml -n web1

curl https://nginx.43.202.54.233.sslip.io/
```

# secret
secret-example.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
data:
  MYSQL_DATABASE: kubernetes
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
stringData:
  MYSQL_ROOT_PASSWORD: "admin1234"

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      name: mysql
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mariadb:10.7
        envFrom:
        - configMapRef:
            name: mysql-config
        - secretRef:
            name: mysql-secret

```
```sh
k apply -f secret-example.yaml 

mysql -uroot -padmin1234
show databases;

+--------------------+
| Database           |
+--------------------+
| information_schema |
| kubernetes         |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
## "kubernetes" database 존재 확인
```
## secretKeyRef 기반으로
secret-example2.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
data:
  MYSQL_DATABASE: kubernetes
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
stringData:
  MYSQL_ROOT_PASSWORD: "admin123456"

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      name: mysql
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mariadb:10.7
        envFrom:
        - configMapRef:
            name: mysql-config
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_ROOT_PASSWORD
```
```
k delete -f secret-example.yaml
k apply -f secret-example2.yaml

mysql -uroot -padmin123456!
show databases;
```
