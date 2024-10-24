# Setting up istio

```
gcloud container clusters get-credentials standard-cluster-1 --zone=europe-west1-b

curl -L https://istio.io/downloadIstio | sh -
cd istio-1.23.2
export PATH=$PWD/bin:$PATH
istioctl version

kubectl get namespace
NAME                 STATUS   AGE

No Istio-system namespace to install it run :
istioctl install

kubectl get namespace
NAME                 STATUS   AGE
istio-system         Active   74s

kubectl get pods -n istio-system
NAME                                    READY   STATUS    RESTARTS   AGE
istio-ingressgateway-7bb6ff9894-krk2p   1/1     Running   0          4m19s
istiod-7b94f8859-6j628                  1/1     Running   0          4m24s

adding labled namespace for default for automatic Envoy Proxy Injection or automatic istio container injection
kubectl label namespace default istio-injection=enabled

kubectl get ns default --show-labels
NAME      STATUS   AGE   LABELS
default   Active   32h   istio-injection=enabled

kubectl get pods --namespace istio-system
kubectl get svc --namespace istio-system

Guidline Youtube vidio URL : https://youtu.be/voAyroDb6xk?si=8_BshvTP9mJBXPhq
```

# Using Kiali

- http://localhost:20001

```
Istio Add-ons website :
https://istio.io/latest/docs/ops/integrations/

run on Google console:

royrobin202@cloudshell:~$ ls
istio-1.23.2  README-cloudshell.txt

inside istio we have folder addon 
PATH : /home/royrobin202/istio-1.23.2/samples/addons

which has all the required addons :
royrobin202@cloudshell:~/istio-1.23.2/samples/addons$ ls
extras  grafana.yaml  jaeger.yaml  kiali.yaml  loki.yaml  prometheus.yaml  README.md

run this to execute all:
kubectl apply -f istio-1.23.2/samples/addons

royrobin202@cloudshell:~$ kubectl get pods -n istio-system
NAME                                    READY   STATUS    RESTARTS   AGE
grafana-7f76bc9cdb-r5g6r                1/1     Running   0          109s
istio-ingressgateway-64f9774bdc-bb7g5   1/1     Running   0          8m4s
istiod-868cc8b7d7-fph29                 1/1     Running   0          8m9s
jaeger-66f9675c7b-q6nvn                 1/1     Running   0          94s
kiali-65c46f9d98-8brhp                  1/1     Running   0          87s
loki-0                                  1/1     Running   0          82s
prometheus-7979bfd58c-vtgwd             2/2     Running   0          78s

royrobin202@cloudshell:~$ kubectl get svc -n istio-system
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                          AGE
grafana                ClusterIP      34.118.234.18    <none>        3000/TCP                                         2m23s
istio-ingressgateway   LoadBalancer   34.118.231.18    34.56.17.66   15021:32509/TCP,80:31431/TCP,443:32236/TCP       53m
istiod                 ClusterIP      34.118.237.149   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP            53m
jaeger-collector       ClusterIP      34.118.229.65    <none>        14268/TCP,14250/TCP,9411/TCP,4317/TCP,4318/TCP   2m6s
kiali                  ClusterIP      34.118.234.62    <none>        20001/TCP,9090/TCP                               2m2s
loki                   ClusterIP      34.118.238.207   <none>        3100/TCP,9095/TCP                                117s
loki-headless          ClusterIP      None             <none>        3100/TCP                                         118s
loki-memberlist        ClusterIP      None             <none>        7946/TCP                                         118s
prometheus             ClusterIP      34.118.232.82    <none>        9090/TCP                                         113s
tracing                ClusterIP      34.118.239.192   <none>        80/TCP,16685/TCP                                 2m8s
zipkin                 ClusterIP      34.118.232.104   <none>        9411/TCP                                         2m7s
```


##### To PORT FORWAD on Local Machin Run :
```
run on Local machine:
kubectl port-forward svc/kiali -n istio-system 30001:20001

you will get :
Forwarding from 127.0.0.1:30001 -> 20001
Forwarding from [::1]:30001 -> 20001

you can access kiali using 127.0.0.1:30001
```

# Using Graphana to see prometheus metrics
- http://localhost:3000

```
kubectl port-forward svc/grafana -n istio-system 3000:3000
```

# Using Jaeger

http://localhost:16686

```
kubectl port-forward -n istio-system $(kubectl get pod -n istio-system -l app=jaeger -o jsonpath='{.items[0].metadata.name}') 16686
```