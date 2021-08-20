# ochacafe-s4-6
Ochacafe4 #6 Observability再入門

デモ環境の構築方法、資材置き場です。

事前に[Kubernetesクラスタ（OKE）の構築](https://docs.oracle.com/ja-jp/iaas/Content/ContEng/Tasks/contengcreatingclusterusingoke.htm)、[OCIR初期セットアップ](https://docs.oracle.com/ja-jp/iaas/Content/Registry/Concepts/registryprerequisites.htm)が完了していることが前提です。

## 環境構築

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

```kubectl
kubectl patch service prometheus -n istio-system -p '{"spec": {"type": "LoadBalancer"}}'
```
```
service/prometheus patched
```

```kubectl
kubectl patch service grafana -n istio-system -p '{"spec": {"type": "LoadBalancer"}}'
```
```
service/grafana patched
```

```kubectl
kubectl patch service kiali -n istio-system -p '{"spec": {"type": "LoadBalancer"}}'
```
```
service/kiali patched
```

```kubectl
kubectl patch service tracing -n istio-system -p '{"spec": {"type": "LoadBalancer"}}'
```
```
service/jaeger-collector patched
```

インストールした内容を確認します。

```kubectl
kubectl get services,deployments -n istio-system -o wide
```
```
NAME                           TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                                                                      AGE   SELECTOR
service/grafana                LoadBalancer   10.56.5.57     34.84.241.xxx    3000:32659/TCP                                                               30m   app.kubernetes.io/instance=grafana,app.kubernetes.io/name=grafana
service/istio-egressgateway    ClusterIP      10.56.0.60     <none>           80/TCP,443/TCP                                                               61m   app=istio-egressgateway,istio=egressgateway
service/istio-ingressgateway   LoadBalancer   10.56.14.58    34.84.61.xxx     15021:32344/TCP,80:31336/TCP,443:30128/TCP,31400:32249/TCP,15443:31226/TCP   61m   app=istio-ingressgateway,istio=ingressgateway
service/istiod                 ClusterIP      10.56.3.178    <none>           15010/TCP,15012/TCP,443/TCP,15014/TCP                                        61m   app=istiod,istio=pilot
service/jaeger-collector       ClusterIP      10.56.1.226    <none>           14268/TCP,14250/TCP,9411/TCP                                                 30m   app=jaeger
service/kiali                  LoadBalancer   10.56.10.122   34.146.209.xxx   20001:30355/TCP,9090:32447/TCP                                               30m   app.kubernetes.io/instance=kiali,app.kubernetes.io/name=kiali
service/prometheus             LoadBalancer   10.56.1.157    34.146.205.xxx   9090:31718/TCP                                                               30m   app=prometheus,component=server,release=prometheus
service/tracing                LoadBalancer   10.56.4.92     35.243.122.xxx   80:31885/TCP,16685:30654/TCP                                                 30m   app=jaeger
service/zipkin                 ClusterIP      10.56.14.192   <none>           9411/TCP                                                                     30m   app=jaeger

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS                                             IMAGES                                                       SELECTOR
deployment.apps/grafana                1/1     1            1           30m   grafana                                                grafana/grafana:7.5.5                                        app.kubernetes.io/instance=grafana,app.kubernetes.io/name=grafana
deployment.apps/istio-egressgateway    1/1     1            1           61m   istio-proxy                                            docker.io/istio/proxyv2:1.11.0                               app=istio-egressgateway,istio=egressgateway
deployment.apps/istio-ingressgateway   1/1     1            1           61m   istio-proxy                                            docker.io/istio/proxyv2:1.11.0                               app=istio-ingressgateway,istio=ingressgateway
deployment.apps/istiod                 1/1     1            1           61m   discovery                                              docker.io/istio/pilot:1.11.0                                 istio=pilot
deployment.apps/jaeger                 1/1     1            1           30m   jaeger                                                 docker.io/jaegertracing/all-in-one:1.23                      app=jaeger
deployment.apps/kiali                  1/1     1            1           30m   kiali                                                  quay.io/kiali/kiali:v1.38                                    app.kubernetes.io/instance=kiali,app.kubernetes.io/name=kiali
deployment.apps/prometheus             1/1     1            1           30m   prometheus-server-configmap-reload,prometheus-server   jimmidyson/configmap-reload:v0.5.0,prom/prometheus:v2.26.0   app=prometheus,component=server,release=prometheus
```

以下ブラウザでアクセスするとWebUIを確認できます。

Prometheus WebUI
http://34.146.205.xxx:9090/

Grafana WebUI
http://34.84.241.xxx:3000/

Kiali WebUI
http://34.146.209.xxx:20001/

Jaeger WebUI
http://35.243.122.xxx/

