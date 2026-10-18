helmchart

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod 700 get_helm.sh
./get_helm.sh

Helm is a package manager for k8s. similar to how apt manages packages on Ubuntu

|concept | Meaning |
|Chart |A package containing Kubernetes templates and default configuration |
|Release | An installed instance of a chart |
|values.yaml | Configuration used to customize a chart |
|Repository | A source containing charts |



helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo bitnami/nginx

sudo kubectl create namespace helm-lab
helm install nginx bitnami/nginx \
  --namespace helm-lab

Error: cannot access to k8s thru localhost:8080
Solution: setup kubeconfig at profile 

mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown "$(id -u):$(id -g)" ~/.kube/config
chmod 600 ~/.kube/config

export KUBECONFIG='/home/Cym_0/.kube/config'
or set at the profile to export the env

now we can install
helm install nginx bitnami/nginx --namespace helm-lab

verification
helm list -n helm-lab
kubectl get all -n helm-lab


Modify on the value
helm get values nginx -n helm-lab
helm show values bitnami/nginx

helm show values bitnami/nginx > values.yamlz
modify the values.yaml
cat > my-values.yaml <<'EOF'
replicaCount: 2
service:
  type: ClusterIP
EOF

helm upgrade nginx bitnami/nginx \
  --namespace helm-lab \
  -f my-values.yaml

  helm list -n helm-lab
  it will update to v2

  kubectl get deployments,pods,svc -n helm-lab

  helm history nginx -n helm-lab

  if there is issue you can rollback:
  helm rollback nginx 1 -n helm-lab


setup ingress.yaml

create nginx-ingress.yaml
kubectl apply -f nginx-ingress.yaml

validate
kubectl get ingress -n helm-lab
kubectl describe ingress nginx-ingress -n helm-lab

testing
forward 
kubectl port-forward -n kube-system svc/traefik 8080:80

curl -H "Host: nginx.lab.local" http://127.0.0.1:8080/


deploy multiple appliactions through Traefik
kubectl create deployment apache \
  --image=httpd:2.4 \
  --namespace=helm-lab
kubectl rollout status deployment/apache -n helm-lab

kubectl expose deployment apache \
  --name=apache \
  --port=80 \
  --target-port=80 \
  --type=ClusterIP \
  --namespace=helm-lab

nano apache-ingress.yaml
kubectl apply -f apache-ingress.yaml


