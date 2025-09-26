# 🚀 Guía de instalación de ArgoCD con Helm

En esta guía se describen los pasos necesarios para desplegar **ArgoCD** en un clúster de Kubernetes utilizando **Helm**.

---

## 📌 Pasos a seguir

### 1️⃣ Crear el namespace de ArgoCD
Crea un espacio de nombres dedicado donde se desplegarán todos los recursos de ArgoCD.

```bash
kubectl create ns argocd
```

---

### 2️⃣ Añadir el repositorio de Argo a Helm
Agrega el repositorio oficial de ArgoCD a Helm y actualiza los índices de charts disponibles.

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

---

### 3️⃣ Instalar ArgoCD con Helm
Instala el chart de ArgoCD en el namespace `argocd` utilizando un archivo `values.yaml` con tu configuración personalizada.  
El flag `--debug` muestra detalles del proceso y `--wait` asegura que Helm espere hasta que todos los pods estén listos.

```bash
helm install argocd ./argo-cd-8.3.7.tgz -n argocd --create-namespace -f values.yaml --debug --wait
```

---

### 4️⃣ Obtener la contraseña inicial de administrador
ArgoCD crea un usuario inicial llamado `admin`. La contraseña se guarda en un **Secret** dentro de Kubernetes.  
Con este comando puedes extraerla y decodificarla.

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

---

### 5️⃣ Ver los nodos del clúster
Comprueba el estado y las características de los nodos donde se está ejecutando Kubernetes.

```bash
kubectl get nodes -o wide
```

---

### 6️⃣ Ver el servicio de ArgoCD
Lista los servicios desplegados en el namespace `argocd`.  
El servicio `argocd-server` será el que te permitirá acceder a la interfaz web de ArgoCD.

```bash
kubectl get svc -n argocd argocd-server
```

---

✅ Con estos pasos, tendrás **ArgoCD instalado y listo para usarse** en tu clúster de Kubernetes.
