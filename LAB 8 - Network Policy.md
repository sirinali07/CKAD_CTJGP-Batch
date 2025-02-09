## Network Policy

### Task 1: Check the network connectivity between pods in different namespaces
Create 2 namespaces
```
kubectl create ns devops
```
Now label the devops namespace and verify it
```
kubectl label ns finance project=myproject
```
```
kubectl get ns finance --show-labels
```
```
kubectl create ns finance
```
Create pod in devops namespace
```
kubectl -n devops run ng-pod --image nginx --port 80 --expose
```
Verify the pod and service in the devops namespace
```
kubectl -n devops get all
```
```
kubectl -n devops get po -o wide
```
Create pod in finance namespace
```
kubectl -n finance run pod2 --image centos -- sleep 7000
```
Verify the pod in the finance namespace
```
kubectl -n devops get po
```

Enter the pod in finance namespace  and check its connectivity with a pod in devops namespace
```
kubectl -n finance exec -it pod2 -- bash
```
Curl with the IP address of ng-pod or service of ng-pod
```
 curl <IP of the ng-pod>
 
```
```
curl <IP of the ng-pod service>
```
```
exit
```
**Note** You can able to curl the ng-pod 

### Task 2: Ingress Network policy with Pod labels 

Create a network policy which uses labels to deny all ingress traffic 
```
vi np-deny-all.yml
```
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  namespace: devops
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      run: ng-pod
  ingress: []
```
```
kubectl apply -f np-deny-all.yml
```
```
kubectl -n devops get networkpolicies
```
```
kubectl -n devops describe networkpolicy backend-policy
```
Now enter into the  pod2 in finance namespace again and verify that it cannot curl the ng-pod from devops namespace

```
kubectl -n finance exec -it pod2 -- bash
```
Curl with the IP address of ng-pod or service of ng-pod
```
 curl <IP of the ng-pod>
```
```
curl <IP of the ng-pod service>
```
It will show timeout
```
exit
```
Modify the network policy to allow traffic with matching namespace labels 
Inspect the network policy and notice the selectors
```
vi np-deny-all.yml
```
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  namespace: devops
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      run: ng-pod
  ingress: 
  - from:
    - namespaceSelector:
        matchLabels:
          project: myproject  
```
Before applying this yaml describe networkpolicies to check rule
```
kubectl -n devops describe networkpolicy backend-policy
```
```
kubectl apply -f np-deny-all.yml
```
```
kubectl -n devops describe networkpolicies backend-policy
```

Now enter into the  pod2 in finance namespace again and verify that it cannot curl the ng-pod from devops namespace

```
kubectl -n finance exec -it pod2 -- bash
```
Curl with the IP address of ng-pod or service of ng-pod
```
 curl <IP of the ng-pod>
```
```
curl <IP of the ng-pod service>
```
You can able to access
```
exit
```
### Task 3: Egress Policy
Create a pod and check the Egress Rules applied to it by default
```
kubectl run pod1 --image nginx
```
```
kubectl exec -it pod1 -- bash
```
```
curl https://8.8.8.8
```
```
curl https://yahoo.com
```
```
exit
```
Create an Egress Policy
```
vi egress-policy.yaml
```
Create a Kubernetes Network Policy manifest file to define egress rules.
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-outbound
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 8.8.8.8/32  # Example IP address
```
Apply the network policy 
```
kubectl apply -f egress-policy.yaml
```
Verify that the egress policy has been applied to your pods:
```
kubectl get networkpolicies
```
```
kubectl describe networkpolicies
```
Enter the Pod1 and check if the network policy has been applied to it
```
kubectl exec -it pod1 -- bash
```
```
curl https://8.8.8.8
```
It can access the IP as it is enabled by the Egress Policy
```
curl https://yahoo.com
```
It is not able to access other than mentioned in the EgressPolicy

```
exit
```
