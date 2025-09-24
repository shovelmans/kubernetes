helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
kubectl create ns argocd
helm install argocd argo/argo-cd -n argocd --create-namespace -f values.yaml --debug
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
kubectl get svc -n argocd
kubectl get nodes -o wide