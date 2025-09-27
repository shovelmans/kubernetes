# Instalación de AWX con AWX Operator (Helm + Kustomize)

## 1. Instalar el AWX Operator con Helm

```bash
helm repo add awx-operator https://ansible-community.github.io/awx-operator-helm/
helm repo update
helm install awx-operator awx-operator/awx-operator -n awx --create-namespace
```

Esto desplegó el operador (`awx-operator-controller-manager`) en el namespace `awx`.

---

## 2. Definir instancia AWX (`awx.yaml`)

```yaml
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
  namespace: awx
spec:
  service_type: NodePort
  nodeport_port: 30080
```

Aplicar los manifiestos:

```bash
kubectl apply -f awx.yaml
kubectl apply -f awx-service.yaml
```

---

## 3. Parche de error en Postgres

El pod de Postgres daba error de permisos en `/var/lib/pgsql/data/userdata`.  
Se resolvió con un parche de `securityContext`:

```bash
kubectl patch statefulset awx-postgres-15   -n awx   --type='json'   -p='[{"op": "add", "path": "/spec/template/spec/securityContext", "value": {"fsGroup": 26}}]'
```

Y luego se eliminó el pod para que se recreara:

```bash
kubectl delete pod awx-postgres-15-0 -n awx
```

---

## 4. Recursos creados por el operador

- Base de datos (`awx-postgres-13-0`)
- Servicios (`awx-service`, `awx-postgres-13`, etc.)
- Pods de aplicación (`awx-task`, `awx-web`)

El pod `awx-web` estaba en **Pending** por falta de memoria, pero el autoscaler añadió nodos y continuó la instalación.

---

## 5. Obtener contraseña del admin

```bash
kubectl get secret awx-admin-password -n awx -o jsonpath="{.data.password}" | base64 --decode; echo
```

---

✅ **En resumen**:  
Los pasos fueron esos 5 que listaste.  
Pero ojo: con **Helm te bastaba** (ya incluye CRDs y operador), lo de `kubectl kustomize` fue redundante.
