
## install helm
```
snap install helm --classic

helm version
```

## rke2 port 확인 
- https://docs.rke2.io/install/requirements
- rke2는 기본적으로 etcd-expose-metrics: false로 되어 있다 
- 따라서 이를 true로 변경 해야 한다 rke2 설치시 /etc/rancher/rke2/config.yaml 에 추가 한다 
```bash
cat <<EOF > /etc/rancher/rke2/config.yaml
write-kubeconfig-mode: "0644"
tls-san:
  - $EXTERNAL_IP
etcd-expose-metrics: true
EOF
```
- rke2 설치후 중간에 설정시 /etc/rancher/rke2/config.yaml 추가후 다음과 같이 재 실행 한다 
```sh
sudo systemctl restart rke2-server.service

```

## install promethues-stack
- kube-prometheus-stack으로 설치시 promethus,grafana,node-exporter가 패키지로 설치됨
```sh
kubectl create ns monitoring
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```
custom-values.yaml
```yaml
kubeControllerManager:  
  service:
    enabled: true
    port: 10250
    targetPort: 10250
kubeScheduler:  
  service:
    enabled: true
    port: 10250
    targetPort: 10250
```
- rke2는  kubelet metric port 10250이다 
```sh
helm install prometheus prometheus-community/kube-prometheus-stack  -f prometheus-custom-values.yaml -n monitoring
```
## access port
- prometheus-kube-prometheus-prometheus  9090
- prometheus-grafana 80
prometheus-ing.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: prometheus-ing
  namespace: monitoring
spec:
  ingressClassName: nginx
  rules:
  - host: "prometheus.3.34.196.90.sslip.io"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: prometheus-kube-prometheus-prometheus 
            port:
              number: 9090

```
grafana-ing.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana-ing
  namespace: monitoring
spec:
  ingressClassName: nginx
  rules:
  - host: "grafana.3.34.196.90.sslip.io"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: prometheus-grafana
            port:
              number: 80
```
## prometheus > status > targets 체크 
-  prometheus ui > status
-  http://prometheus.3.34.196.90.sslip.io
-  모두 UP 상태가 되어야 한다 
  
## grafana UI
- http://grafana.3.34.196.90.sslip.io
- admin/prom-operator
- datasource 및 dashboard가 이미 설정및 설치 되어 있다 

## clear
```bash
helm uninstall prometheus -n monitoring
```

# opensearch

## worker 3개 필요 
- opensearch 설치 하기 위해서 최소한 worker가 3개 필요하다 

## pv 때문에 설치(Rancher Local Path Provisioner)
```sh
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.26/deploy/local-path-storage.yaml

```

```sh
kubectl create ns monitoring
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install opensearch bitnami/opensearch -f opensearch-custom-values.yaml -n monitoring 
 
```
- running 할때까지 시간이 걸림

## dashboard ingress
dashboard-ing.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: opensearch-dashboard-ing
  namespace: monitoring
spec:
  ingressClassName: nginx
  rules:
  - host: "dashboard.3.34.196.90.sslip.io"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: opensearch-dashboards 
            port:
              number: 5601

```

## sidecar log 수집
```sh
kubectl create ns nginx
kubectl apply -f exam5/sidcar-nginx-log.yaml

```
- nginx ui 접속
- http://nginx.3.34.196.90.sslip.io/

## opensearch dashboard에서 확인 
- index management > Indices >  server-nginx-log-* 확인
- Dashboards Management > Index patterns > Create index pattern > server-nginx-log-* 설정

## clear 
```sh 
kubectl delete -f exam5/sidcar-nginx-log.yaml
```

## cluster-level 로그 수집 

### nginx deployment
```bash
kubectl apply -f exam5/nginx.yaml
```

### install fluent-bit daemonset
- fluent-bit 을 daemonset을 배포한다 
```sh
helm install fluent-bit bitnami/fluent-bit -f exam5/fluentbit-daemonset-custom-values.yaml --create-namespace  --namespace monitoring

```
### log 확인 
- index management > Indices >  nginx-* 확인

## clear 
```sh
helm uninstall opensearch -n monitoring
helm uninstall fluent-bit -n monitoring
## pvc 를 강제적으로 삭제한다 
kubectl get pvc -n monitoring 

kubectl delete -f exam5/nginx.yaml
```
