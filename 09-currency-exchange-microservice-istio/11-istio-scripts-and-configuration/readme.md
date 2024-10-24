# Setting up istio

```
gcloud container clusters get-credentials standard-cluster-1 --zone=europe-west1-b

kubectl create namespace istio-system
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.23.2
export PATH=$PWD/bin:$PATH
istioctl version
istioctl install --set profile=default
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
helm template istio-base istio/base --namespace istio-system > istio-base.yaml
helm template istiod istio/istiod --set global.mtls.enabled=false --set tracing.enabled=true --set kiali.enabled=true --set grafana.enabled=true --namespace istio-system > istio.yaml
kubectl apply -f istiod.yaml
kubectl apply -f istio-base.yaml
kubectl get pods --namespace istio-system
kubectl get svc --namespace istio-system
kubectl label namespace default istio-injection=enabled
```

# Using Kiali

- http://localhost:20001

```
kubectl port-forward \
    $(kubectl get pod -n istio-system -l app=kiali \
    -o jsonpath='{.items[0].metadata.name}') \
    -n istio-system 20001
```
Create a secret

```
kubectl get secret -n istio-system kiali
kubectl create secret generic kiali -n istio-system --from-literal=username=admin --from-literal=passphrase=admin
```

# Using Graphana to see prometheus metrics
- http://localhost:3000

```
kubectl -n istio-system port-forward \
    $(kubectl -n istio-system get pod -l app=grafana \
    -o jsonpath='{.items[0].metadata.name}') 3000
```

# Using Jaeger

http://localhost:16686

```
kubectl port-forward -n istio-system \
    $(kubectl get pod -n istio-system -l app=jaeger \
    -o jsonpath='{.items[0].metadata.name}') 16686
```