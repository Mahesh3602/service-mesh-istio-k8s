
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

# 11. Accessing external application with SA
  #### create SA account for access to soecific ns
 kubectl apply -f security-layer/default-namespace-authorization-policies.yaml

 #### create sleep pod to use this SA account
 Kubectl apply -f security-layer/sleep-with-productpage-sa.yaml
 
 #### to check certs- not stored on disk, distributed securely using sds(secret discovery service)
 kubectl logs -l app=productpage -c istio-proxy

++++++++++++++++++++++++++++++++++++

# 12. as part of security principal only product svc should access review.. all other svc should  by default deny. so , apply deny all by default on reviews.
 ## check & deny
  ### to check
  ```
kubectl describe peerauthentication

kubectl get authorizationpolicy
```
  ### to apply
 kubectl apply -f security-layer/reviews-deny-all.yaml

 ## Allow access from product page. It uses service account
  ### to check
  ```
kubectl get serviceaccount

kubectl get po -l app=productpage -o json | jq '.items[0].spec.serviceAccountName'
```
  ### apply authorisation policy
kubectl apply -f security-layer/reviews-allow-productpage.yaml
  
  ### test

kubectl exec -it deploy/sleep -- sh
```

Try accessing the reviews & ratings APIs:

```
curl http://reviews:9080/1

curl http://ratings:9080/ratings/1
``

# 13. configure for all service (like step 12)
kubectl apply -f security-layer/defaultNS-workload-authorization-policy.yaml

 ### test
 ```
kubectl exec -it deploy/sleep -- sh

curl http://reviews:9080/1

curl http://ratings:9080/ratings/1

curl http://details:9080/details/1

curl http://productpage:9080
```
# 14. step 13 , can also be obtained with coarse-grained model, targeting whole NS.
  ### delete existing implementation
  ```
kubectl delete authorizationpolicy --all

kubectl apply -f demo2/bookinfo-namespace-authorization-policies.yaml
```

```
kubectl exec -it deploy/sleep -- curl http://ratings:9080/ratings/1
```
 ### apply coarse grained model
kubectl apply -f security-layer/default-namespace-authorization-policy.yaml

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++










