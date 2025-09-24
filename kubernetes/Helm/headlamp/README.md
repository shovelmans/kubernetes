helm repo add headlamp https://kubernetes-sigs.github.io/headlamp/
helm repo update
kubectl create namespace headlamp 
helm upgrade headlamp headlamp/headlamp -n headlamp -f values.yaml
kubectl apply -f headlamp-rbac.yaml
kubectl create token headlamp-admin -n headlamp
