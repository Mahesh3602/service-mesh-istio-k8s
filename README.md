
# Prerequistes
- kubernetes cluster
- loadbalncer controller installed to host and check apps

# 1. get istio helm repos
helm repo add istio https://istio-release.storage.googleapis.com/charts

helm repo update

helm search repo istio

# 2. setup the base, which sets up all the custom resource definitions (CRDs):
helm install istio-base istio/base -n istio-system --create-namespace

# 3. Install istio controlplane components
helm install istiod istio/istiod -n istio-system --wait

helm install istiod istio/istiod -n istio-system \
  --set resources.requests.memory=512Mi \
  --set resources.requests.cpu=100m

# 4. finally install the gateway, which will take care of routing traffic from outside the cluster into our Pods:
kubectl create ns istio-ingress
helm install istio-ingress istio/gateway -n istio-ingress

helm install istio-ingress istio/gateway -n istio-ingress \
  --set "resources.requests.memory=256Mi" \
  --set "resources.requests.cpu=100m" 

   ## check
helm ls -A

# 5. Deploy sample app to mesh after enbling isto-injection
kubectl label namespace default istio-injection=enabled

kubectl apply -f sampleApp/bookinfo.yaml

-- each pod should have 2 containers
-- load alb url

++++++++++++++++++++

# 6 install virtual service
 kubectl apply -f virtualService/details-virtualservice.yaml 

 ### just to try routing of traffic
 kubectl apply -f virtualService/details-virtualservice-retry.yaml 

+++++++++++++++++++++

# 7. Deploying one of the service(reviews) and managing traffic through Istio

kubectl apply -f deploy-v2-review/reviews-v2.yaml 
kubectl get endpoints -l app=reviews 

# 9. Lets install gateway with vs
kubectl apply -f deploy-v2-review/03-bookinfo-gateway.yaml 


# 8. Installing Virtualservice with destination rules and see the switch of traffic
kubectl apply -f deploy-v2-review/destinationRule-reviews.yaml

+++++++++++++++++++++++

# 9. security relatd

## Run a shell in the Product Page Pod container to access details page:

```
kubectl exec -it deploy/productpage-v1 -- python
```

```
import urllib.request
urllib.request.urlopen("http://details:9080/details/1").read()
exit()
```

 ## install legacy application
 kubectl apply -f security-layer/sleep-in-legacy-ns.yaml

 ## Run a shell in the legacy Pod container and acces details svc

```
kubectl -n legacy get po

kubectl -n legacy exec -it deploy/sleep -- sh
```

Use the details API:

```
nslookup details.default.svc.cluster.local

curl http://details.default.svc.cluster.local:9080/details/1

curl http://details.default.svc.cluster.local:9080/details/100
```

# 10. enforce TLS
  ### Enforce mTLS for all services in the default namespace:

kubectl apply -f security-layer/mutual-tls.yaml
   ### run all the commands from stp 9 again.


