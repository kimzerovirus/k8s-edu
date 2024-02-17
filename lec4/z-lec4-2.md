# lecture-4
- install-vm에서 실행 
- ubuntu유저로  실행   
```sh
# cd ~
# git clone https://github.com/io203/k8s-edu.git
cd  k8s-edu/lec4
```

# 1. emptyDir
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.17
        ports:
        - containerPort: 80

        volumeMounts:
        - name: log-volume
          mountPath: /var/log/nginx

      - name: fluent-bit
        image: amazon/aws-for-fluent-bit:2.1.0
        volumeMounts:
        - name: log-volume
          mountPath: /var/log/nginx

      volumes: # 볼륨 선언
      - name: log-volume
        emptyDir: {}
```
```sh 

k apply -f emptydir-vol.yaml

## emptyDir volume인 log-volume으로 설정해 놓았기 때문에 
##  fluent-bit container에서 이제 nginx 의 access.log  error.log 를 읽을수 있도록 가능해 졌다 
## nginx pod의 fluent-bit container로 접속하여 아래와 같이  nginx의 로그파일이 조회 되는지 확인한다 
ls /var/log/nginx

## clear 
k delete -f emptydir-vol.yaml
```

# 2. hostPath
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-webserver
spec:
  containers:
  - name: test-hostpath-nginx
    image: nginx:1.17
    volumeMounts:
    - mountPath: /var/local/aaa
      name: mydir
    - mountPath: /var/local/aaa/1.txt
      name: myfile
  volumes:
  - name: mydir
    hostPath:
      # 파일 디렉터리가 생성되었는지 확인한다.
      path: /var/local/aaa
      type: DirectoryOrCreate
  - name: myfile
    hostPath:
      path: /var/local/aaa/1.txt
      type: FileOrCreate
```
```sh
## nginx를 배포한다 
k apply -f hostpath-vol.yaml

## pod의 디렉토리및 파일이 생성 되었는지 확인  
k exec -it test-webserver -- ls /var/local
k exec -it test-webserver -- ls /var/local/aaa

## 실제 pod가 배포된 node의 에서 디렉토리및 파일이 생성 되었는지 확인  
##  pod가 배포된 노드 확인 
k get pod test-webserver -o wide
## node에 ubuntu로 로그인 하여 host에  생성 되었는지  조회 한다 
ls /var/local/aaa

## 다른 노드에서 확인한다 
## ls /var/local/aaa 조회 되지 않을 것이다 

## clear 
k delete -f hostpath-vol.yaml
```

# 3. pv/pvc

## 3.1  노드에 index.html 파일 생성
```sh
# 사용자 노드에서 슈퍼유저로 명령을 수행하기 위하여
# "sudo"를 사용한다고 가정한다
## worker-1
sudo mkdir /mnt/data
sudo sh -c "echo 'Hello from Kubernetes storage' > /mnt/data/index.html"
cat /mnt/data/index.html
```

## pv
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"

```
```sh
kubectl apply -f task-pv-volume.yaml
kubectl get pv task-pv-volume
```
## pvc
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```
```sh
kubectl apply -f https://k8s.io/examples/pods/storage/pv-claim.yaml

kubectl get pvc task-pv-claim
kubectl get pv task-pv-volume
```

## pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```
```sh
kubectl apply -f https://k8s.io/examples/pods/storage/pv-pod.yaml
kubectl get pod task-pv-pod

kubectl exec -it task-pv-pod -- /bin/bash

apt update
apt install curl
curl http://localhost/
```
### clean 
```sh
kubectl delete pod task-pv-pod
kubectl delete pvc task-pv-claim
kubectl delete pv task-pv-volume
```

# storageClass

## nfs storageClass 예
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-client
provisioner: k8s-sigs.io/nfs-subdir-external-provisioner # or choose another name, must match deployment's env PROVISIONER_NAME'
parameters:
  pathPattern: "${.PVC.namespace}/${.PVC.annotations.nfs.io/storage-path}" # waits for nfs.io/storage-path annotation, if not specified will accept as empty string.
  onDelete: delete
```
## pvc example
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-claim
  annotations:
    nfs.io/storage-path: "test-path" # not required, depending on whether this annotation was shown in the storage class description
spec:
  storageClassName: nfs-client
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Mi

```

## Rancher Local Path Provisioner

```sh
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.26/deploy/local-path-storage.yaml

kubectl -n local-path-storage get pod
## log
kubectl -n local-path-storage logs -f -l app=local-path-provisioner

```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-path-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path
  resources:
    requests:
      storage: 128Mi
```
```sh
kubectl create -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/examples/pvc/pvc.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
spec:
  containers:
  - name: volume-test
    image: nginx:stable-alpine
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: volv
      mountPath: /data
    ports:
    - containerPort: 80
  volumes:
  - name: volv
    persistentVolumeClaim:
      claimName: local-path-pvc
```
```sh
kubectl create -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/examples/pod/pod.yaml

kubectl get pv                
## pvc-9875c93c-bb64-4761-b588-2b92341156af   128Mi     

kubectl get pvc
## local-path-pvc   Bound    pvc-9875c93c-bb64-4761-b588-2b92341156af   128Mi 
kubectl get pod

kubectl exec volume-test -- sh -c "echo local-path-test > /data/test"

# delete pod
kubectl delete -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/examples/pod/pod.yaml

# recreate
kubectl create -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/examples/pod/pod.yaml

# check volume content
kubectl exec volume-test -- sh -c "cat /data/test"
```

## clear
```
kubectl delete -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.26/deploy/local-path-storage.yaml
```

## longhorn
```sh
## root 에서 실행 
## jq 설치
snap install jq

## longhorn check
curl -sSfL https://raw.githubusercontent.com/longhorn/longhorn/v1.6.0/scripts/environment_check.sh | bash


## longhorn-iscsi  설치해 준다 
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.6.0/deploy/prerequisite/longhorn-iscsi-installation.yaml



## nfs client설치해 준다 
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.6.0/deploy/prerequisite/longhorn-nfs-installation.yaml

## longhorn check
curl -sSfL https://raw.githubusercontent.com/longhorn/longhorn/v1.6.0/scripts/environment_check.sh | bash

## install longhorn
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.6.0/deploy/longhorn.yaml

kubectl -n longhorn-system get pod

```

## longhorn 예제 
```sh
kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-longhorn-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: longhorn
  resources:
    requests:
      storage: 128Mi
EOF

```
```yaml
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
spec:
  containers:
  - name: volume-test
    image: nginx:stable-alpine
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: volv
      mountPath: /data
    ports:
    - containerPort: 80
  volumes:
  - name: volv
    persistentVolumeClaim:
      claimName: test-longhorn-pvc
EOF
```
```sh
kubectl exec volume-test -- sh -c "echo ====local-path-test========== > /data/test"

kubectl delete pod volume-test 

kubectl exec volume-test -- sh -c "cat /data/test"

```
## longhorn UI
```yaml
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: longhorn-ing
  namespace: longhorn-system
spec:
  ingressClassName: nginx
  rules:
  - host: "longhorn.3.34.50.226.sslip.io"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: longhorn-frontend
            port:
              number: 80
EOF
```
- UI 접속: http://longhorn.3.34.50.226.sslip.io
## longhorn uninstall
```sh
## Please set it to `true` using Longhorn UI or kubectl -n longhorn-system edit settings.longhorn.io deleting-confirmation-flag 
## ui에서 Deleting Confirmation Flag: 체크박스 체크 한다


kubectl create -f https://raw.githubusercontent.com/longhorn/longhorn/v1.6.0/uninstall/uninstall.yaml
kubectl get job/longhorn-uninstall -n longhorn-system -w

kubectl delete -f https://raw.githubusercontent.com/longhorn/longhorn/v1.6.0/deploy/longhorn.yaml
kubectl delete -f https://raw.githubusercontent.com/longhorn/longhorn/v1.6.0/uninstall/uninstall.yaml
```


# Request / Limit

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: images.my-company.example/app:v4
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
  - name: log-aggregator
    image: images.my-company.example/log-aggregator:v6
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"


```

# ETCD 
```sh
kubectl -n kube-system exec -it etcd-ip-172-26-15-174 -- /bin/bash

etcd --version
etcdctl version

export ETCDCTL_ENDPOINTS='https://127.0.0.1:2379' 
export ETCDCTL_CACERT='/var/lib/rancher/rke2/server/tls/etcd/server-ca.crt' 
export ETCDCTL_CERT='/var/lib/rancher/rke2/server/tls/etcd/server-client.crt' 
export ETCDCTL_KEY='/var/lib/rancher/rke2/server/tls/etcd/server-client.key' 
export ETCDCTL_API=3 



etcdctl member list
etcdctl endpoint status --cluster -w table
etcdctl endpoint health

```
