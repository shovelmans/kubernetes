# Instalación de AWX con AWX Operator (Helm + Kustomize)

## 1. Instalar el AWX Operator con Helm

```bash
helm repo add awx-operator https://ansible-community.github.io/awx-operator-helm/
helm repo update
helm install awx-operator awx-operator/awx-operator -n awx --create-namespace
```

Esto desplegó el operador (`awx-operator-controller-manager`) en el namespace `awx`.

---

## 2. Aplicar los CRDs con Kustomize

```bash
kubectl kustomize . | kubectl apply -f -
```

Esto creó los CRDs (`awx.awx.ansible.com`, etc.) que permiten definir un recurso de tipo **AWX**.

👉 Aquí se mezcló **Helm + Kustomize**, pero como los CRDs y el operador son compatibles, no se estorbaron.

---

## 3. Definir instancia AWX (`awx.yaml`)

Archivo `awx.yaml`:

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

Archivo `awx-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: awx-service
  namespace: awx
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: awx-web
    app.kubernetes.io/part-of: awx
  ports:
    - port: 8052       # Puerto dentro del cluster
      targetPort: 8052 # Puerto del contenedor
      nodePort: 30080  # Puerto expuesto en el nodo
```

Aplicar los manifiestos:

```bash
kubectl apply -f awx.yaml
kubectl apply -f awx-service.yaml
```

---

## 4. Parche de error en Postgres

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

## 5. Recursos creados por el operador

- Base de datos (`awx-postgres-13-0`)
- Servicios (`awx-service`, `awx-postgres-13`, etc.)
- Pods de aplicación (`awx-task`, `awx-web`)

El pod `awx-web` estaba en **Pending** por falta de memoria, pero el autoscaler añadió nodos y continuó la instalación.

---

## 6. Obtener contraseña del admin

```bash
kubectl get secret awx-admin-password -n awx -o jsonpath="{.data.password}" | base64 --decode; echo
```

---

✅ **En resumen**:  
Los pasos fueron esos 5 que listaste.  
Pero ojo: con **Helm te bastaba** (ya incluye CRDs y operador), lo de `kubectl kustomize` fue redundante.
