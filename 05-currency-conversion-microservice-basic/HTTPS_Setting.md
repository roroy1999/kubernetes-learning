Reffer : https://cloud.google.com/dns/docs/set-up-dns-records-domain-name

search in Google Cloud -> Load balancing

under Load balancing -> select Load balancers Tab

and select the load balancer there -> (in my case) k8s2-um-qovxf9c2-default-gateway-ingress-10n7r5tq -> Click EDIT

inside it : 
under : Frontend configuration -> click Add Frontend IP and port ->

Do the Below configuration :

Name : agro-manipal-https-ip
Protocol : HTTPS (includes HTTP/2 and HTTP/3)
Network Service Tier : Standard (us-central1)
IP version : IPv4
IP address : Ephemeral
Port : 443
Certificate : my-ssl-cert
if not present create one using below step :

```
Name : my-ssl-cert

Create mode : Create Google-managed certificate

Domains
Domain 1 : agromanipalfood.shop
Domain 2 : www.agromanipalfood.shop
```

SSL policy : GCP default
HTTP/3 (QUIC) negotiation : Automatic (default)

-> Done

-> outside click Save or Update


### To CREATE STATIC EXTERNAL IP:

search -> VPC networks

on the right select -> IP addresses

select -> Reserve External static IP address ->

name : my-global-ip (give any name but remember it since same will be used in ingress)

Network Service Tier : Premium

IP version : IPv4

Type : Regional(based on your availability need can also be global)

Region : us-central1 (Iowa)

Attached to : gke-robin-cluster-default-pool-dff86c35-84rf

-> Reserve

### Adding Global IP to Ingress:

inside ingress add kubernetes.io/ingress.regional-static-ip-name: "my-global-ip" under annotation:

ex: 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: gateway-ingress
  annotations:
    kubernetes.io/ingress.regional-static-ip-name: "my-global-ip"
    
- run-> kubectl apply -f ingress.yaml


