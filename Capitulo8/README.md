# Práctica 8. Gestión de namespaces y despliegue de recursos en Kubernetes

## Objetivo de la práctica:
Al finalizar la práctica, serás capaz de:
- Comprender el concepto de namespaces en Kubernetes y su importancia en la organización y gestión de recursos dentro de un clúster.
- Aprender a crear y utilizar namespaces para desplegar y administrar diferentes tipos de objetos en Kubernetes.
- Implementar recursos clave de Kubernetes, incluyendo ReplicaSets, Jobs, DaemonSets y Deployments, dentro de namespaces específicos.

## Duración aproximada:
- 90 minutos.

---

**[⬅️ Atrás](https://netec-mx.github.io/DOCK_KUB/Capitulo7/)** | **[Lista General](https://netec-mx.github.io/DOCK_KUB/)** | **[Siguiente ➡️](https://netec-mx.github.io/DOCK_KUB/Capitulo9/)**

---

## Instrucciones:

- Crear un nuevo namespace en Kubernetes, aprendiendo a segmentar el clúster en espacios lógicos para una mejor organización y seguridad.
- Desplegar un objeto de tipo ReplicaSet en el namespace recién creado, aprendiendo a gestionar y escalar aplicaciones en Kubernetes.
- Crear un nuevo namespace en Kubernetes, aprendiendo a segmentar el clúster en espacios lógicos para una mejor organización y seguridad.

### Tarea 1. Crear proyecto Node JS.

Paso 1. Crea una carpeta con el nombre `mi-proyecto-nodejs` para inicializar el proyecto.

```bash
mkdir mi-proyecto-nodejs
cd mi-proyecto-nodejs
npm init -y
```

![cap8_start_node.png](../images/cap8_start_node.png)

Paso 2. Crea el archivo `index.js` y agrega el siguiente código:

```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;
app.get('/', (req, res) => {
res.send('Hola Mundo desde Node.js!');
});
app.listen(PORT, () => {
console.log(`Servidor corriendo en puerto ${PORT}`);
});
```

Paso 3. Instala **express**.

```bash
npm install express
```

![cap8_start_express.png](../images/cap8_start_express.png)

Paso 4. Crear un archivo Dockerfile.

```Dockerfile
FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```

![cap8_docker_file.png](../images/cap8_docker_file.png)

Paso 5. Sube la imagen a Docker Hub.

```bash
docker build -t daniel0223/netec_docker_repo:v1 .
docker push daniel0223/netec_docker_repo:v1
```

![cap8_up_docker_hub.png](../images/cap8_up_docker_hub.png)


### Tarea 2. Desplegar en Kubernetes.
Paso 1. Crea el namespace con el nombre `namespace.yaml`.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: mi-namespace
```

![cap8_namespace.png](../images/cap8_namespace.png)

Paso 2. Aplica y crea el namespace.

```bash
kubectl apply -f namespace.yaml
```

![cap8_comand_namespace.png](../images/cap8_comand_namespace.png)

Paso 3. Crea el archivo `deployment.yaml`.

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mi-deployment
  namespace: mi-namespace
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mi-app-nodejs
  template:
    metadata:
      labels:
        app: mi-app-nodejs
    spec:
      containers:
        - name: mi-app-nodejs
          image: daniel0223/netec_docker_repo:v1
          ports:
            - containerPort: 3000
```

![cap8_deployment_yml.png](../images/cap8_deployment_yml.png)

Paso 4. Aplica el **deployment**.

```bash
kubectl apply -f deployment.yaml
```

![cap8_deployment_yml_exec.png](../images/cap8_deployment_yml_exec.png)

Paso 5.  Crea el **ReplicaSet** y ejecútalo: 

```yaml
# replicaset.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mi-replicaset
  namespace: mi-namespace
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mi-app-nodejs
  template:
    metadata:
      labels:
        app: mi-app-nodejs
    spec:
      containers:
        - name: mi-app-nodejs
          image: daniel0223/netec_docker_repo:v1
          ports:
            - containerPort: 3000

```

```bash
kubectl -f replicaset.yaml apply
```

![cap8_k8s_k8s.png](../images/cap8_k8s_k8s.png)

Paso 6. Creación del **StatefulSet** y ejecución:

```yaml
# statefulset.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mi-replicaset
  namespace: mi-namespace
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mi-app-nodejs
  template:
    metadata:
      labels:
        app: mi-app-nodejs
    spec:
      containers:
        - name: mi-app-nodejs
          image: daniel0223/netec_docker_repo:v1
          ports:
            - containerPort: 3000

```

```bash
kubectl -f statefulset.yaml apply
```

Paso 7. Creación **DaemonSet**, y ejecución: 
```yaml
# daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: mi-daemonset
  namespace: mi-namespace
spec:
  selector:
    matchLabels:
      app: mi-app-nodejs
  template:
    metadata:
      labels:
        app: mi-app-nodejs
    spec:
      containers:
        - name: mi-app-nodejs
          image: daniel0223/netec_docker_repo:v1
          ports:
            - containerPort: 3000
```

```bash
kubectl -f statefulset.yaml apply
```
![cap8_deamonSet.png](../images/cap8_deamonSet.png)

Paso 8. Verifica que la configuración esté completa.

```yaml
kubectl get all -n mi-namespace
```

![cap8_final.png](../images/cap8_final.png)

## Resultado esperado:

![cap8_final.png](../images/cap8_final.png)
