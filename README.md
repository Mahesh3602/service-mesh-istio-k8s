
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

# 5. Deploy sample app to mesh
kubectl label namespace default istio-injection=enabled

kubectl create ns bookinginfo
kubectl label namespace bookinginfo istio-injection=enabled

kubectl apply -f sampleApp/bookinfo.yaml -n bookinginfo

-- each pod should have 2 containers
-- load alb url

++++++++++++++++++++

# 6 install virtual service
 kubectl apply -f virtualService/details-virtualservice.yaml -n bookinginfo

 ### just to try routing of traffic
 kubectl apply -f virtualService/details-virtualservice-retry.yaml -n bookinginfo

+++++++++++++++++++++

# 7. Deploying one of the service(reviews) and managing traffic through Istio

kubectl apply -f deploy-v2-review/reviews-v2.yaml --namespace=bookinginfo
kubectl get endpoints -l app=reviews -n bookinginfo

# 8. Installing Virtualservice with destination rules and see the switch of traffic
kubectl apply -f deploy-v2-review/50-50-review.yaml

+++++++++++++++++++++++

# 9. Lets install gateway with vs
kubectl apply -f deploy-v2-review/03-bookinfo-gateway.yaml 


