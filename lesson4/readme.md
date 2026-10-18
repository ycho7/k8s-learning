ArgoCD GitOps

kubectl create namespace argocd
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl apply --server-side --force-conflicts \
  -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

nano guestbook-app.yaml
kubectl apply -f guestbook-app.yaml

kubectl get applications -n argocd
kubectl describe application guestbook -n argocd

kubectl port-forward -n argocd svc/argocd-server 8443:443

login https://localhost:8443


kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
echo


kubectl get application guestbook -n argocd \
  -o jsonpath='{.status.sync.status}{" / "}{.status.health.status}{"\n"}'
OutOfSync / Missing

go to the website and click sync and it will auto provisoning pods for the gitops for the repo 

manually update the deployment, it will out of sync, if it does not do automatically sync, it wont auto sync 

update the guest-app.yaml 
```
    automated:
      selfHeal: true
      prune: false
```
testing the drift again
kubectl scale deployment guestbook-ui \
  --replicas=3 \
  -n gitops-demo

this time when u create the new pods, it will auto terminating the created pods

