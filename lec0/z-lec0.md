

# install rke2
- single-master : lec0 > single-master 
- multi-master : lec0 > multi-master 

# uninstall rke2
```sh
rke2-uninstall.sh
```

## 2.2 k9s 설치 
```bash
# install k9s with snap
snap install k9s 
ln -s /snap/k9s/current/bin/k9s /snap/bin/
```


# 3. 로컬에서 kubectl remote 접속 하기 

```sh
## master-1에서 실행 
cat ~/.kube/config

## 로컬위치(~/.kube/config) 복사해 넣는다 
export KUBECONFIG=~/.kube/config

# master-1의 external-ip로 변경한다  
# kubectl get pod -A 로 하면 접속이 안되어 timeout이 된다 
# lightsail master01의 방화벽에서 6443 포트를 open 한다 

kubectl get pod -A
kubectl get nodes
```

# 4 nginx 배포 
```
kubectl create ns nginx
kubectl create deployment nginx --image=nginx -n nginx
```


## 참고: tls-san 변경하거나 추가 한다면
```bash
# 수정한다 
vi  /etc/rancher/rke2/config.yaml

# 재 시작해 주면 된다 
sudo systemctl restart rke2-server.service

sudo systemctl status  rke2-server.service
```

