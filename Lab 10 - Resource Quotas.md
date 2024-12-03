## Resource Quotas

### Task 1: Creating a Namespace

In Kubernetes, a ResourceQuota is a way to limit the amount of resources (CPU, memory, and persistent storage) that a namespace can consume. It helps in preventing resource exhaustion and ensures that one namespace doesn't negatively impact the performance or stability of others.

Keep in mind that ResourceQuotas are a way to define constraints, and they do not actively enforce them on existing resources. They are checked when new resources are created or when existing resources are updated. If a namespace exceeds its quota, the creation or update of resources may be rejected.

The below command can identify all namespaced objects
```
kubectl api-resources
```
```
kubectl create namespace ns1
```
```
kubectl get ns
```
```
kubectl describe ns ns1
```
```
kubectl describe quota -n ns1
```


### Task 2: Creating Resource Quota and Constraining Object Creation

Create a pod and expose it, before applying the resource quota to check if the resource quota applies to existing objects or not.
```
kubectl -n ns1 run pod1 --image nginx --port 80
```
```
kubectl -n ns1 expose pod pod1 --name pod1-svc --port 80 --type NodePort 
```


**Imperative**
```
kubectl -n ns1 create quota rs-quota1 --hard=pods=2,services=1
```
```
kubectl describe ns ns1
```
```
kubectl get quota -n ns1
```
**Declarative** (OR)
```
vi rq1.yaml
```
```yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: rs-quota1
  namespace: ns1
spec:
  hard:
    pods: "2"
    services: "1"

```
```
kubectl apply -f rq1.yaml
```
**Note- Either create Quota with **Imperative Way** OR **Declarative Way**
```
kubectl describe ns ns1
```
```
kubectl get quota -n ns1
```
```
kubectl describe quota -n ns1
```
```
kubectl -n ns1 run pod2 --image nginx --port 80
```
```
kubectl -n ns1 expose pod pod2 --name pod2-svc --port 80 --type NodePort 
```
Try to deploying a new pod in namespace ns1
```
kubectl -n ns1 run pod3 --image nginx --port 80
```
**Once the the desire quota acheaved, It will not allow you to exceed the limit and you will get Frobidden Message.
##### Delete the quota & pods created in previous steps
```
kubectl delete quota rs-quota1 -n ns1
```
```
kubectl -n ns1 delete pod --all
```

### Task 3: Creating Resource Quota and Constraining Hardware Resources

```
vi rq2.yaml
```
```yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: rs-quota2
  namespace: ns1
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    pods: "2"
    services: "1"
    count/deployments.apps: "1"
```
```
kubectl apply -f rq2.yaml
```
```
kubectl describe ns ns1
```
```
kubectl describe quota -n ns1
```
### Task 4: Verify Resource Quota Functionality
```
kubectl -n ns1 run pod4 --image nginx --port 80
```
```
vi pod5.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod5
  namespace: ns1
spec:
  containers:
  - name: quota-ctr
    image: nginx
    resources:
      limits:
        memory: "800Mi"
        cpu: "1000m"
      requests:
        memory: "600Mi"
        cpu: "350m"
    ports:
      - containerPort: 80
```
```	  
kubectl apply -f pod5.yaml
```
```
kubectl describe ns ns1
```
```
kubectl describe quota -n ns1
```
```
kubectl get resourcequota -n ns1 -o yaml
```
Deploy another Pod, such that the total resource exceeds the resoucre quota limit
```
vi pod6.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod6
  namespace: ns1
spec:
  containers:
  - name: quota-ctr
    image: nginx
    resources:
      limits:
        memory: "1600Mi"
        cpu: "2000m"
      requests:
        memory: "1600Mi"
        cpu: "800m"
    ports:
      - containerPort: 80
```
```
kubectl apply -f pod6.yaml
```

