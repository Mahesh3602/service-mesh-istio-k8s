
# Traffic Management
##​ The VirtualService defines how traffic is routed to the microservices within the mesh. While Kubernetes Services provide basic load balancing, Istio’s Virtual Service allows for fine-grained traffic control.  
​Key Components:
​
## this Virtual Service allows the mesh to perform 
* header-based routing
* retries
* traffic shifting without changing application code.  

##​ Why this is used:
- ​Even in this simple configuration, the VirtualService acts as a placeholder for advanced operational tasks:
* simple routing.
  - demonstrated in step-6 using details vs
* Timeout Policies: Setting strict request deadlines to prevent cascading failures across the Bookinfo app.
  - demonstrated in step-7 using __ vs
* Fault Injection: Simulating 503 errors or delays to test the resilience of the productpage.
  - demonstrated in step-8 using __ vs
* ​Canary Revisions: Easily split traffic between v1 and v2 of the details service.  
  - demonstrated in step-9 using review vs 











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
