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

# 9. route traffic 50% traffic to new review verion
kubectl apply -f deploy-v2-review/virtualService-split-traffic-review.yaml
