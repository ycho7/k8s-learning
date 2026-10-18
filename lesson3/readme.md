HTTPS and TLS with Traefik on K3s

openssl req -x509 -nodes -days 30 \
  -newkey rsa:2048 \
  -keyout nginx.key \
  -out nginx.crt \
  -subj "/CN=nginx.lab.local" \
  -addext "subjectAltName=DNS:nginx.lab.local"

create nginx-https-ingress.yaml
kubectl apply -f nginx-https-ingress.yaml

kubectl port-forward -n kube-system svc/traefik 8443:443

curl -v \
  --cacert nginx.crt \
  --resolve nginx.lab.local:8443:127.0.0.1 \
  https://nginx.lab.local:8443/

Setup automated TLS 
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set crds.enabled=true

kubectl get all -n cert-manager

Create a self-signer issuer
nano lab-issuer.yaml
kubectl apply -f lab-issuer.yaml

Request a certificate automatically
nano nginx-certificate.yaml
kubectl apply -f nginx-certificate.yaml

kubectl describe certificate nginx-lab-certificate -n helm-lab

Tell Traefik to use the automated certificate
Edit nginx-https-ingress.yaml
```
tls:
  - hosts:
      - nginx.lab.local
    secretName: nginx-tls-auto
```

check validitiy of the certificate
kubectl get secret nginx-tls-auto -n helm-lab \
  -o jsonpath='{.data.tls\.crt}' |
  base64 -d |
  openssl x509 -noout -dates -issuer -subject

