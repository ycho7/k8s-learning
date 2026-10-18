Install k3s 

curl -sfL https://get.k3s.io | sh -

To verify 
```
kubectl get nodes 
```

show k8s api server address
```
sudo kubectl cluster-info
```

Deploy first application
```
### Create a Deployment with two replicas
sudo kubectl create deployment nginx --image=nginx:stable --replicas=2

### Check the pods
sudo kubectl get deployments
sudo kubectl get pods -o wide

### expose nginx inside the cluster
sudo kubectl expose deployment nginx --port=80 --type=ClusterIP

### Verify the Service
sudo kubectl get svc nginx
```

Test for k8s self-healing
```
sudo kubectl get pods
sudo kubectl delete pod <pod-name>

### u can use this to live trace 
sudo kubectl get pods -w

watch -n 1 'sudo kubectl get pods'
```

To scale up
```
sudo kubectl scale deployment nginx --replicas=3
```

Learn Kubernetes Services
```
sudo kubectl get svc
sudo kubectl describe svc nginx
sudo kubectl get endpoints nginx
```


Create namespace
```
sudo kubectl get namespaces
sudo kubectl create namespace learning
sudo kubectl create deployment nginx \
  --image=nginx:stable \
  --replicas=2 \
  -n learning
sudo kubectl get pods -n learning

### expose it 
sudo kubectl expose deployment nginx \
  --port=80 \
  --type=ClusterIP \
  -n learning


sudo kubectl get svc -A
sudo kubectl get deploy -A
sudo kubectl get ns
```

ConfigMap 
```
sudo kubectl create configmap app-config \
  --from-literal=APP_NAME=k8s-learning \
  --from-literal=ENVIRONMENT=development

sudo kubectl get configmap app-config -o jsonpath='{.data.APP_NAME}'
sudo kubectl get configmap app-config -o jsonpath='{.data.ENVIRONMENT}'
```

results:
```
Cym_0@instance-ycho-learning:~$ sudo kubectl describe configmap app-config
Name:         app-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
APP_NAME:
----
k8s-learning

ENVIRONMENT:
----
development


BinaryData
====

Events:  <none>
```

use it inside a pods
```
sudo kubectl run config-test \
  --image=busybox:1.36 \
  --restart=Never \
  --env="APP_NAME=$(sudo kubectl get configmap app-config -o jsonpath='{.data.APP_NAME}')" \
  --env="ENVIRONMENT=$(sudo kubectl get configmap app-config -o jsonpath='{.data.ENVIRONMENT}')" \
  --command -- sh -c 'echo "App: $APP_NAME"; echo "Environment: $ENVIRONMENT"; sleep 3600'
```

Result:
```
Cym_0@instance-ycho-learning:~$ sudo kubectl logs config-test
App: k8s-learning
Environment: development
```

Update the configmap
```
sudo kubectl patch configmap app-config --type merge -p '{"data":{"ENVIRONMENT":"staging"}}'
sudo kubectl get configmap app-config -o yaml
```
then delete and reapply the pods

kubernetes secrets
sudo kubectl create secret generic app-secret \
  --from-literal=DB_USERNAME=labuser \
  --from-literal=DB_PASSWORD='demo-password-123'

sudo kubectl get secrets

sudo kubectl run secret-test \
  --image=busybox:1.36 \
  --restart=Never \
  --env="DB_USERNAME=$(sudo kubectl get secret app-secret -o jsonpath='{.data.DB_USERNAME}' | base64 -d)" \
  --env="DB_PASSWORD=$(sudo kubectl get secret app-secret -o jsonpath='{.data.DB_PASSWORD}' | base64 -d)" \
  --command -- sh -c 'echo "Username: $DB_USERNAME"; echo "Password is configured: ${DB_PASSWORD:+yes}"; sleep 3600'

sudo kubectl logs secret-test


From larger organizational scope to smaller workload units, it's:
Namespace → Deployment → ReplicaSet → Pods → Containers
hierarchy check got do
sudo kubectl get deployments,replicasets,pods -n default

