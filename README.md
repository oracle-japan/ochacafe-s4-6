# ochacafe-s4-6
Ochacafe4 #6 Observability再入門

デモ環境の構築方法、資材置き場です。

## 環境構築

### Kubernetesクラスタの構築

Quick Create Clusterを利用して構築

* Name: cluster1
* Kubernetes version: v1.20.8
* Kubernetes API Endpoint: Public Endpoint
* Kubernetes Worker Nodes: Public Workers
* Shape: VM.Standard.E3 Flex vCpu 2 Mem 32
* Nuber of nodes: 3

### Istio Install (addon install(Prometheus,Grafana,Kiali,Jaeger))

`Istio 1.11.0` をインストールします。

```bash
ISTIO_VERSION=1.11.0
```

```bash
curl -L https://istio.io/downloadIstio | ISTIO_VERSION="${ISTIO_VERSION}" sh -
```
```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   102  100   102    0     0    213      0 --:--:-- --:--:-- --:--:--   212
100  4549  100  4549    0     0   6425      0 --:--:-- --:--:-- --:--:--  6425

Downloading istio-1.11.0 from https://github.com/istio/istio/releases/download/1.11.0/istio-1.11.0-linux-amd64.tar.gz ...

Istio 1.11.0 Download Complete!

Istio has been successfully downloaded into the istio-1.11.0 folder on your system.

Next Steps:
See https://istio.io/latest/docs/setup/install/ to add Istio to your Kubernetes cluster.

To configure the istioctl client tool for your workstation,
add the /home/yutaka_ich/istio-1.11.0/bin directory to your environment path variable with:
         export PATH="$PATH:/home/yutaka_ich/istio-1.11.0/bin"

Begin the Istio pre-installation check by running:
         istioctl x precheck 

Need more information? Visit https://istio.io/latest/docs/setup/install/ 
```

istioctlコマンドを実行できるようにパスを通します。

```bash
export PATH="${PWD}/istio-${ISTIO_VERSION}/bin:${PATH}"
```
```bash
echo PATH="\"${PWD}/istio-${ISTIO_VERSION}/bin:\${PATH}\"" >> ~/.bashrc
```

Istioのバージョンを確認します。

```istioctl
istioctl version
```
```
no running Istio pods in "istio-system"
1.11.0
```

Istioのコンポーネントをインストールします。

```istioctl
istioctl install --set profile=demo --skip-confirmation
```
```
✔ Istio core installed                                                                                                                           
✔ Istiod installed                                                                                                                               
✔ Egress gateways installed                                                                                                                      
✔ Ingress gateways installed                                                                                                                     
✔ Installation complete                                                                                                                          
Thank you for installing Istio 1.11.  Please take a few minutes to tell us about your install/upgrade experience!  https://forms.gle/kWULBRjUv7hHci7T6
```

必要なアドオン（Prometheus,Grafana,Kiali,Jaegerなど）をインストールします。

```kubectl
kubectl apply -f "istio-${ISTIO_VERSION}/samples/addons/"
```
```
serviceaccount/grafana created
configmap/grafana created
service/grafana created
deployment.apps/grafana created
configmap/istio-grafana-dashboards created
configmap/istio-services-grafana-dashboards created
deployment.apps/jaeger created
service/tracing created
service/zipkin created
service/jaeger-collector created
serviceaccount/kiali created
configmap/kiali created
clusterrole.rbac.authorization.k8s.io/kiali-viewer created
clusterrole.rbac.authorization.k8s.io/kiali created
clusterrolebinding.rbac.authorization.k8s.io/kiali created
role.rbac.authorization.k8s.io/kiali-controlplane created
rolebinding.rbac.authorization.k8s.io/kiali-controlplane created
service/kiali created
deployment.apps/kiali created
serviceaccount/prometheus created
configmap/prometheus created
clusterrole.rbac.authorization.k8s.io/prometheus created
clusterrolebinding.rbac.authorization.k8s.io/prometheus created
service/prometheus created
deployment.apps/prometheus created
```

Prometheus、Grafana、Kiali、JaegerのWebUIにブラウザからアクセスできるようにします。

```
kubectl patch service prometheus -n istio-system -p '{"spec": {"type": "NodePort"}}'
```
```
service/prometheus patched
```
```
kubectl patch service grafana -n istio-system -p '{"spec": {"type": "NodePort"}}'
```
```
service/grafana patched
```
```
kubectl patch service kiali -n istio-system -p '{"spec": {"type": "NodePort"}}'
```
```
service/kiali patched
```
```
kubectl patch service tracing -n istio-system -p '{"spec": {"type": "NodePort"}}'
```
```
service/tracing patched
```

```
kubectl get services,deployments -n istio-system -o wide
```
```
NAME                           TYPE           CLUSTER-IP      EXTERNAL-IP         PORT(S)                                                                      AGE   SELECTOR
service/grafana                NodePort       10.96.96.80     <none>              3000:31746/TCP                                                               21m   app.kubernetes.io/instance=grafana,app.kubernetes.io/name=grafana
service/istio-egressgateway    ClusterIP      10.96.110.50    <none>              80/TCP,443/TCP                                                               22m   app=istio-egressgateway,istio=egressgateway
service/istio-ingressgateway   LoadBalancer   10.96.16.94     xxx.xxx.xxx.xxx     15021:32007/TCP,80:32374/TCP,443:32343/TCP,31400:30388/TCP,15443:31719/TCP   22m   app=istio-ingressgateway,istio=ingressgateway
service/istiod                 ClusterIP      10.96.235.244   <none>              15010/TCP,15012/TCP,443/TCP,15014/TCP                                        22m   app=istiod,istio=pilot
service/jaeger-collector       ClusterIP      10.96.121.97    <none>              14268/TCP,14250/TCP,9411/TCP                                                 21m   app=jaeger
service/kiali                  NodePort       10.96.182.15    <none>              20001:31964/TCP,9090:31854/TCP                                               21m   app.kubernetes.io/instance=kiali,app.kubernetes.io/name=kiali
service/prometheus             NodePort       10.96.238.223   <none>        9090:31147/TCP                                                               21m   app=prometheus,component=server,release=prometheus
service/tracing                NodePort       10.96.216.254   <none>        80:32418/TCP,16685:31229/TCP                                                 21m   app=jaeger
service/zipkin                 ClusterIP      10.96.184.172   <none>        9411/TCP                                                                     21m   app=jaeger

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS                                             IMAGES                                                       SELECTOR
deployment.apps/grafana                1/1     1            1           21m   grafana                                                grafana/grafana:7.5.5                                        app.kubernetes.io/instance=grafana,app.kubernetes.io/name=grafana
deployment.apps/istio-egressgateway    1/1     1            1           22m   istio-proxy                                            docker.io/istio/proxyv2:1.11.0                               app=istio-egressgateway,istio=egressgateway
deployment.apps/istio-ingressgateway   1/1     1            1           22m   istio-proxy                                            docker.io/istio/proxyv2:1.11.0                               app=istio-ingressgateway,istio=ingressgateway
deployment.apps/istiod                 1/1     1            1           22m   discovery                                              docker.io/istio/pilot:1.11.0                                 istio=pilot
deployment.apps/jaeger                 1/1     1            1           21m   jaeger                                                 docker.io/jaegertracing/all-in-one:1.23                      app=jaeger
deployment.apps/kiali                  1/1     1            1           21m   kiali                                                  quay.io/kiali/kiali:v1.38                                    app.kubernetes.io/instance=kiali,app.kubernetes.io/name=kiali
deployment.apps/prometheus             1/1     1            1           21m   prometheus-server-configmap-reload,prometheus-server   jimmidyson/configmap-reload:v0.5.0,prom/prometheus:v2.26.0   app=prometheus,component=server,release=prometheus
```

OCIダッシュボードから、[Networking]-[Virtual Cloud Networks]を選択して対象のVCNを選択します。

「oke-vcn-quick-cluster1-xxxxxxxxx-regional」を選択します。

「oke-nodesubnet-quick-cluster1-xxxxxxxxx-regional」を選択します。

「Add Ingress Rules」ボタンをクリックします。

以下を設定して、「Add Ingress Rules」ボタンをクリックします。

SOURCE CIDR: 0.0.0.0/0  
Destination Port Range: 30000-65535

NodeのEXTERNAL-IPを確認します。

```
kubectl get nodes -o wide
```
```
NAME          STATUS   ROLES   AGE     VERSION   INTERNAL-IP   EXTERNAL-IP       OS-IMAGE                  KERNEL-VERSION                    CONTAINER-RUNTIME
10.0.10.133   Ready    node    7m13s   v1.20.8   10.0.10.133   150.136.xxx.xxx    Oracle Linux Server 7.9   5.4.17-2102.203.6.el7uek.x86_64   cri-o://1.20.2
10.0.10.179   Ready    node    7m23s   v1.20.8   10.0.10.179   132.145.xxx.xxx    Oracle Linux Server 7.9   5.4.17-2102.203.6.el7uek.x86_64   cri-o://1.20.2
10.0.10.84    Ready    node    7m19s   v1.20.8   10.0.10.84    129.213.xxx.xxx    Oracle Linux Server 7.9   5.4.17-2102.203.6.el7uek.x86_64   cri-o://1.20.2
```

Prometheus,Grafana,Kiali,JaegerのNodePortを確認します。

```
kubectl get services -n istio-system
```
```
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP         PORT(S)                                                                      AGE
grafana                NodePort       10.96.96.80     <none>              3000:31746/TCP                                                               25m
istio-egressgateway    ClusterIP      10.96.110.50    <none>              80/TCP,443/TCP                                                               25m
istio-ingressgateway   LoadBalancer   10.96.16.94     xxx.xxx.xxx.xxx     15021:32007/TCP,80:32374/TCP,443:32343/TCP,31400:30388/TCP,15443:31719/TCP   25m
istiod                 ClusterIP      10.96.235.244   <none>              15010/TCP,15012/TCP,443/TCP,15014/TCP                                        25m
jaeger-collector       ClusterIP      10.96.121.97    <none>              14268/TCP,14250/TCP,9411/TCP                                                 25m
kiali                  NodePort       10.96.182.15    <none>              20001:31964/TCP,9090:31854/TCP                                               25m
prometheus             NodePort       10.96.238.223   <none>              9090:31147/TCP                                                               25m
tracing                NodePort       10.96.216.254   <none>              80:32418/TCP,16685:31229/TCP                                                 25m
zipkin                 ClusterIP      10.96.184.172   <none>              9411/TCP
```

以下ブラウザでアクセスします。NodeのEXTERNAL-IPは3個のうちどれでも大丈夫です。

* Prometheus: 150.136.xxx.xxx:31147
* Grafana: 150.136.xxx.xxx:31746
* Kiali: 150.136.xxx.xxx:31964
* Jaeger: 150.136.xxx.xxx:32418

### node exporter Install

node exporterのDaemonSetを作成します。

```
kubectl apply -f https://raw.githubusercontent.com/oracle-japan/ochacafe-s4-6/main/manifests/node-exporter-handson.yaml
```
```
serviceaccount/node-exporter-handson created
service/node-exporter-handson created
daemonset.apps/node-exporter-handson created
```

node-exporter-handsonというPodが3個稼働していることを確認します。

```
kubectl get pods
```
```
NAME                           READY   STATUS    RESTARTS   AGE
node-exporter-handson-7x7m7   1/1     Running   0          53s
node-exporter-handson-hbjnd   1/1     Running   0          53s
node-exporter-handson-nzd4l   1/1     Running   0          53s
```

### Grafana Loki Install & Set up

#### Install

```
helm repo add grafana https://grafana.github.io/helm-charts
```
```
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/yutaka_ich/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /home/yutaka_ich/.kube/config
"grafana" has been added to your repositories
```

```
helm repo update
```
```
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/yutaka_ich/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /home/yutaka_ich/.kube/config
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "argo" chart repository
...Successfully got an update from the "grafana" chart repository
Update Complete. ⎈Happy Helming!⎈
```

```
helm upgrade --install loki --namespace=istio-system grafana/loki-stack
```
```
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/yutaka_ich/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /home/yutaka_ich/.kube/config
Release "loki" does not exist. Installing it now.
NAME: loki
LAST DEPLOYED: Sun Aug 22 07:22:15 2021
NAMESPACE: istio-system
STATUS: deployed
REVISION: 1
NOTES:
The Loki stack has been deployed to your cluster. Loki can now be added as a datasource in Grafana.

See http://docs.grafana.org/features/datasources/loki/ for more detail.
```

「Loki-0」、「loki-promtail-xxxxx」(3個)がRunningであることを確認します。


```
kubectl get pods -n istio-system
```
```
NAME                                    READY   STATUS    RESTARTS   AGE
grafana-556f8998cd-bkrw8                1/1     Running   0          36m
istio-egressgateway-9dc6cbc49-rv9ll     1/1     Running   0          37m
istio-ingressgateway-7975cdb749-tk4rf   1/1     Running   0          37m
istiod-77b4d7b55d-tq7hh                 1/1     Running   0          37m
jaeger-5f65fdbf9b-28v7w                 1/1     Running   0          36m
kiali-787bc487b7-jkc22                  1/1     Running   0          36m
loki-0                                  1/1     Running   0          2m46s
loki-promtail-lzxg5                     1/1     Running   0          2m46s
loki-promtail-rlrq2                     1/1     Running   0          2m46s
loki-promtail-s7rfz                     1/1     Running   0          2m46s
prometheus-9f4947649-c7swm              2/2     Running   0          36m
```

#### Set up

Grafanaダッシュボードを開いて、左メニューの[Configuration]-[Data Sources]を選択します。

「Add data source」ボタンをクリックします。

「Logging & document databases」にある「Loki」にカーソルを合わせて「Select」ボタンをクリックします。

Lokiの設定画面の「URL」に「http://loki:3100/」と入力して、「Save & Test」ボタンをクリックします。

左メニューの「Explore」を選択して、プルダウンメニューのLokiを選択できれば完了です。

### BookInfo Application Install

istio-ingressgatewayのEXTERNAL-IPとPort番号を変数化します。

```
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
```
```
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
```

さらにGATEWAY_URLとしてINGRESS_HOSTとINGRESS_PORTを変数化します。

```
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
```

自動サイドカーインジェクションのラベルを付与します。

```
kubectl label namespace istio-system istio-injection=enabled
```

BookInfoアプリケーションをインストールします。

```
kubectl apply -n istio-system -f https://raw.githubusercontent.com/istio/istio/release-1.11/samples/bookinfo/platform/kube/bookinfo.yaml
```
```
service/details created
serviceaccount/bookinfo-details created
deployment.apps/details-v1 created
service/ratings created
serviceaccount/bookinfo-ratings created
deployment.apps/ratings-v1 created
service/reviews created
serviceaccount/bookinfo-reviews created
deployment.apps/reviews-v1 created
deployment.apps/reviews-v2 created
deployment.apps/reviews-v3 created
service/productpage created
serviceaccount/bookinfo-productpage created
deployment.apps/productpage-v1 created
```

以下のServiceが稼働していることを確認します。

```
kubectl get services -n istio-system
```
```
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                                                                      AGE
istio-ingressgateway   LoadBalancer   10.96.66.68     146.56.xxx.xxx   15021:31788/TCP,80:30987/TCP,443:31448/TCP,31400:32379/TCP,15443:31984/TCP   4h29m
ratings                ClusterIP      10.96.124.47    <none>           9080/TCP                                                                     31s
reviews                ClusterIP      10.96.231.88    <none>           9080/TCP                                                                     28s
・
・
・
```

以下のPodが稼働していることを確認します。

```
kubectl get pods -n istio-system
```
```
NAME                                    READY   STATUS    RESTARTS   AGE
details-v1-79f774bdb9-89gdb             2/2     Running   0          54s
productpage-v1-6b746f74dc-mds9n         2/2     Running   0          38s
ratings-v1-b6994bb9-q4tk4               2/2     Running   0          50s
reviews-v1-545db77b95-lfmmf             2/2     Running   0          46s
reviews-v2-7bf8c9648f-rwx6p             2/2     Running   0          45s
reviews-v3-84779c7bbc-h4kmw             2/2     Running   0          43s
・
・
・
```

gatewayを作成します。

```
kubectl apply -n istio-system -f https://raw.githubusercontent.com/istio/istio/release-1.11/samples/bookinfo/networking/bookinfo-gateway.yaml
```
```
gateway.networking.istio.io/bookinfo-gateway created
virtualservice.networking.istio.io/bookinfo created
```

bookinfo-gatewayが作成されたことを確認します。

```
kubectl get gateway -n istio-system
```
```
NAME               AGE
bookinfo-gateway   28s
```

実際にcurlで接続確認します。

```
curl -s "http://${GATEWAY_URL}/productpage" | grep -o "<title>.*</title>"
```
```
<title>Simple Bookstore App</title>
```

ブラウザを起動して、BookInfoアプリケーションに接続します。istio-ingressgatewayのEXTERNAL-IP

http://146.56.xxxx.xxx/productpage

![BookInfo](image/ochacafe-s4-6-01.png "BookInfo App")

