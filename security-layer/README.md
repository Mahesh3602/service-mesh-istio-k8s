
# Application Security

## 9. The default application PeerAuthentication setup for Istio is permissive mTLS mode. 
- Encrypted and accessible within mesh. 
- Accessible out of mesh

#### Run a shell in the Product Page Pod container to access details page.
-  Pod is part of mesh and  managed through Istio, It will use mutual TLS, you get response back from below(Test-1) commands.
- Test:1
    * kubectl exec -it deploy/productpage-v1 -- python3 -c 'import urllib.request; print(urllib.request.urlopen("http://details:9080/details/1").read())'

#### Run a shell in the legacy Pod container(outside of mesh) and acces details svc
- install legacy application outside of mesh
*  kubectl apply -f security-layer/sleep-in-legacy-ns.yaml
- Currently the permission is not set to STRICT. you get response from below(Test-2) commands
- Test:2
    * kubectl -n legacy exec deploy/sleep -- nslookup details.default.svc.cluster.local
    * kubectl -n legacy exec deploy/sleep -- curl -s http://details.default.svc.cluster.local:9080/details/1
    * kubectl -n legacy exec deploy/sleep -- curl -s -o /dev/null -w "HTTP Status: %{http_code}\n" http://details.default.svc.cluster.local:9080/details/1

## 10. Enforce TLS - to lock step 9
### Enforce mTLS for all services in the default namespace

* kubectl apply -f security-layer/application/mutual-tls.yaml
    - mtls conection is changed to strict
    - Run Test:1 command of step 9. Accessible from within the pod
    - Runt Test:2 commnds of step 9. Not accessible

## 11. Accessing external application with Service Account
- By default, Istio uses the identity of the Service Account in the Pod as the secure name in the TLS certificate
    * kubectl get serviceaccount
    * kubectl get po -l app=productpage -o json | jq '.items[0].spec.serviceAccountName'

### Lets deny all access to review service except productpage authorisation.

- Apply to deny all access to review svc 
    * kubectl apply -f security-layer/application/reviews-deny-all.yaml
- Below command(test:3) should deny access likely 403 error.
- Test:3
    * kubectl exec -it deploy/productpage-v1 -- python3 -c 'import urllib.request; print(urllib.request.urlopen("http://reviews:9080/reviews/1").read())'
    * On url load - you should see error at the review display

- Allow access using Producbookinfo-productpage SA
    * kubectl apply -f security-layer/application/reviews-allow-productpage.yaml
- Repeat Test:3 . This should succeed.

- Deploy the legacy container in the default namespace try accessing details(success) and review(No-success)
    * kubectl apply -f security-layer/application/sleep-in-default-ns.yaml
    * kubectl exec deploy/sleep -- curl -sI http://reviews:9080/1 | grep HTTP  
    * kubectl exec deploy/sleep -- curl -sI http://details:9080/details/1 | grep HTTP   
    
### security for external applications
- Authorization rules depend on the Service Account of the Pod. If a user has access to the resources, they can deploy an app using a permitted Service Account
- Above result can be obtained with a coarse-grained model, targetting the whole namespace
    * kubectl apply -f security-layer/application/sa-default-namespace-authorization-policies.yaml
- Deploy sleep2 app with SA of productpage.
    * kubectl apply -f security-layer/application/sleep-with-productpage-sa.yaml


# End-user Authorization
## 12. Apply the JWT authentication policy for the product page
    * kubectl apply -f security-layer/user/productpage-authn-jwt.yaml
- execute below command(Test:4) 
- Test:4
    -  RBAC access denied
    * curl -v http://52.150.38.57/productpage

    - try with JWT Token: 401, JWT issuer not configured 
    * curl -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c' -v http://localhost/productpage

    - try with valid token: 200
    * curl -H 'Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IkRIRmJwb0lVcXJZOHQyenBBMnFYZkNtcjVWTzVaRXI0UnpIVV8tZW52dlEiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjQ2ODU5ODk3MDAsImZvbyI6ImJhciIsImlhdCI6MTUzMjM4OTcwMCwiaXNzIjoidGVzdGluZ0BzZWN1cmUuaXN0aW8uaW8iLCJzdWIiOiJ0ZXN0aW5nQHNlY3VyZS5pc3Rpby5pbyJ9.CfNnxWP2tcnR9q0vxyxweaF3ovQYHYZl82hAUsn21bwQd9zP7c-LS9qd_vpdLG4Tn1A15NxfCjp5f7QNBUo-KC9PJqYpgGbaXhaGx7bEdFWjcwv3nZzvc7M__ZpaCERdwU7igUmJqYGBYQ51vr2njU9ZimyKkfDe3axcyiBZde7G6dabliUosJvvKOPcKIWPccCgefSj_GNfwIip3-SsFdlR7BtbVUcqR-yv-XOxJ3Uc1MI0tz3uMiiZcyPV7sNCU4KRnemRIMHVOfuvHsU60_GhGbiSFzgPTAa9WTltbnarTbxudb_YEOx12JiwYToeX0DCPb43W1tzIBxgm8NxUg' -v http://localhost/productpage

## 13. Apply an authorization policy which allows access by issuer & Subject
    * kubectl apply -f security-layer/user/productpage-authz-allow-issuer.yaml
    * kubectl apply -f security-layer/user/productpage-authz-allow-subject.yaml
    * kubectl apply -f security-layer/user/productpage-authz-allow-claim.yaml

## 14. Integration with third-party auth
-- productpage-authz-azure.yaml

