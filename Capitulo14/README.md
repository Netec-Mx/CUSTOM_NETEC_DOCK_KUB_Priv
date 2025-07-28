# Manual paso a paso implementando k8s 

# Paso 1 Crear Artefactos de la aplicación

Para crear el artefacto del back ejecutar el siguiente comando en la terminal:

```bash
 ./gradlew clean build
```
![1-1-create-artefact.png](./images/1-1-create-artefact.png)


Para verificar que se ha creado el artefacto, revisar la carpeta `build/libs` del proyecto. Debería aparecer un archivo con el nombre `contact-agenda-0.0.1-SNAPSHOT.jar`.

![1-2-verify-artifact.png](./images/1-2-verify-artifact.png)

# Paso 2 Dockerizando BackEnd

## 1 Crear DockerFile

1. Click derecho en la carpeta del proyecto y crear un archivo llamado `Dockerfile`.

![2-1-create-File-DockerFile.png](./images/2-1-create-File-DockerFile.png)

2. Crear el contenido en el archivo `Dockerfile`:

```dockerfile
# Imagen base recomendada para Java 17 en producción/K8s
FROM eclipse-temurin:17-jdk-alpine

# Crea el directorio de trabajo
WORKDIR /app

# Copia el artefacto jar generado por Gradle
COPY build/libs/contact-agenda-0.0.1-SNAPSHOT.jar app.jar

# Expone el puerto del backend
EXPOSE 8080

# Comando de inicio
ENTRYPOINT ["java", "-jar", "app.jar"]
```

3. Crear imagen basado en docker file
```bash
 docker build -t contact-agenda-backend .
```
![2-3-create-image-back-springboot.png](./images/2-3-create-image-back-springboot.png)
4. Crear contenedor basado en la imagen creada
```bash
 docker run -p 8080:8080 contact-agenda-backend
```
![2-4-create-container.png](./images/2-4-create-container.png)
5. Validamos en la siguiente URL si se desplego correctaente
   http://localhost:8080/swagger-ui/index.html#/API%20contact/create
![2-5-verify-container-back-end.png](./images/2-5-verify-container-back-end.png)

# Paso 3 Habilitar Kubernetes en Docker Desktop

1. Abrir Docker Desktop y dirigirse a la sección de "Settings" (Configuración).
![3-1-docker-desktop-settings.png](./images/3-1-docker-desktop-settings.png)
2. En la pestaña "Kubernetes", activar la opción "Enable Kubernetes" (Habilitar Kubernetes).
![3-2-docker-enable-k8s.png](./images/3-2-docker-enable-k8s.png)
3. Hacer clic en "Apply & Restart" (Aplicar y Reiniciar) para aplicar los cambios y reiniciar Docker Desktop.
![3-3-reset-docker-desktop-k8s.png](./images/3-3-reset-docker-desktop-k8s.png)
4. Una vez que Docker Desktop se haya reiniciado, verificar que Kubernetes esté funcionando correctamente. Debería aparecer un mensaje indicando que Kubernetes está activo.
![3-4-install-k8s.png](./images/3-4-install-k8s.png)
5. Para verificar que Kubernetes está funcionando correctamente, abrir una terminal y ejecutar el siguiente comando:
```bash
kubectl cluster-info
```
Debería mostrar información sobre el clúster de Kubernetes, incluyendo la dirección del servidor de la API y otros detalles.

![3-5-verify-terminal-k8s.png](./images/3-5-verify-terminal-k8s.png)

# Paso 4 Configurar Kubernetes para el proyecto
1. Crear la carpeta `k8s` en la raíz del proyecto.

![4-1-create-file-k8s.png](./images/4-1-create-file-k8s.png)

2. Se creara el archivo namespace.yaml con el siguiente contenido:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: contact-app
```

![4-2-create-namespace-yaml.png](./images/4-2-create-namespace-yaml.png)

3. Crear el namespace ejecutando el siguiente comando en la terminal:

```bash
kubectl apply -f k8s/namespace.yaml
```
![4-3-create-namespace-terminal.png](./images/4-3-create-namespace-terminal.png)

4. Verificar que el namespace se haya creado correctamente:

```bash
kubectl get namespaces
```

![4-4-verify-namespace-created.png](./images/4-4-verify-namespace-created.png)

5. Se Creara el Secret lo cual permite almacenar de forma segura las credenciales de la base de datos. Crear el archivo `secret.yaml` con el siguiente contenido:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: contact-app
type: Opaque
data:
  mysql-root-password: bmV0ZWN0MTIz
```

![4-5-create-secret-k8s.png](./images/4-5-create-secret-k8s.png)

6. Crear el Secret ejecutando el siguiente comando en la terminal:

```bash
kubectl apply -f k8s/secret.yaml
```
![4-6-create-secret-terminal.png](./images/4-6-create-secret-terminal.png)
7. Verificar que el Secret se haya creado correctamente:

```bash
kubectl get secrets -n contact-app
```

![4-7-verify-secret-created.png](./images/4-7-verify-secret-created.png)

# Nota : Las variables en el Secret deben estar codificadas en base64. Para codificar una variable, puedes usar el siguiente comando en la terminal:

```bash
echo -n 'valor' | base64
```

# Paso 5 Creacion y persistencia de Base de Datos MySQL (Volumen persistente + Deployment para MySQL.)

1. Crear el archivo 'mysql-volume.yaml' para garantiza que tu base de datos no pierda datos si el contenedor se reinicia.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
  namespace: contact-app
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

![5-1-create-mysql-volume.png](./images/5-1-create-mysql-volume.png)

2. Para crear el Deployment de MySQL, crear el archivo `mysql-deployment.yaml` con el siguiente contenido:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: contact-app
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:5.7
          imagePullPolicy: IfNotPresent
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: mysql-root-password
          ports:
            - containerPort: 3306
          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql
      volumes:
        - name: mysql-storage
          persistentVolumeClaim:
            claimName: mysql-pvc
```

![5-2-create-mysql-deployment.png](./images/5-2-create-mysql-deployment.png)

3. Crear el Service para exponer MySQL internamente dentro de Kubernetes. Crear el archivo `mysql-service.yaml` con el siguiente contenido:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: contact-app
spec:
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  type: ClusterIP
```

![5-3-create-mysql-service.png](./images/5-3-create-mysql-service.png)

4. Crear el volumen persistente ejecutando el siguiente comando en la terminal:

```bash
kubectl apply -f k8s/mysql-volume.yaml
```
![5-4-create-mysql-volume-terminal.png](./images/5-4-create-mysql-volume-terminal.png)

5. Crear el Deployment de MySQL ejecutando el siguiente comando en la terminal:

```bash
kubectl apply -f k8s/mysql-deployment.yaml
```

![5-5-create-mysql-deployment-terminal.png](./images/5-5-create-mysql-deployment-terminal.png)

6. Crear el Service de MySQL ejecutando el siguiente comando en la terminal:

```bash
kubectl apply -f k8s/mysql-service.yaml
```

![5-6-create-mysql-service-terminal.png](./images/5-6-create-mysql-service-terminal.png)

7. Verificar que el Deployment de MySQL se haya creado correctamente:

```bash
kubectl get pods -n contact-app
```

![5-7-verify-mysql-deployment.png](./images/5-7-verify-mysql-deployment.png)

8.   verificaremos que el svc de MySQL se haya creado correctamente:
```bash
kubectl get svc -n contact-app
```
![5-8-verify-mysql-service.png](./images/5-8-verify-mysql-service.png)

9. Para poder conectarnos conectarnos a la base de datos con dbever ejecutamos el siguiente comando en la terminal:

```bash
kubectl port-forward svc/mysql 3306:3306 -n contact-app
```
![5-9-port-forward-mysql.png](./images/5-9-port-forward-mysql.png)

10. Abrimos DBeaver y creamos una nueva conexión MySQL con los siguientes datos:

![5-10-create-dbeaver-connection.png](./images/5-10-create-dbeaver-connection.png)

11. Crearemos la base de datos `db-contact` en la base de datos:

![5-11-create-database-dbeaver.png](./images/5-11-create-database-dbeaver.png)

![5-11-1-create-database-dbeaver.png](./images/5-11-create-database-dbeaver.png)


# Paso 6: Desplegar el backend (Spring Boot) en Kubernetes

1. Se debe ajustar el archivo application.yaml del backend para que se conecte a la base de datos MySQL. Asegúrate de que el archivo `application.yaml` tenga la siguiente configuración:
1.1. la url debe quedar de la siguiente manera:
   url: jdbc:mysql://mysql-service:3306/db-contact
1.2  Ajustar la contraela como una variable :
   password: ${MYSQL_PASSWORD:netect123}

```yaml
server:
   port: 8080
spring:
   datasource:
      url: jdbc:mysql://mysql:3306/db-contact?useSSL=false&serverTimezone=UTC
      username: root
      password: ${MYSQL_PASSWORD:netect123}
      driver-class-name: com.mysql.cj.jdbc.Driver

   jpa:
      hibernate:
         ddl-auto: update
      show-sql: true
      properties:
         hibernate:
            dialect: org.hibernate.dialect.MySQL8Dialect
springdoc:
   api-docs:
      path: /api-docs
   show-actuator: false
   packages-to-scan: com.netec.contact.agenda.controller
```
![6-1-back-properties.png](./images/6-1-back-properties.png)

2. Crear el artefacto del back con el comando 
```bash
./gradlew clean build -x test
```
![6-2-back-create-artefact.png](./images/6-2-back-create-artefact.png)

3. Crear la imagen de Docker del backend ejecutando el siguiente comando en la terminal:

```bash
docker build -t contact-agenda-backend:latest .
```
![6-3-back-create-docker-img.png](./images/6-3-back-create-docker-img.png)


3.Verificar que la imagen de Docker del backend se haya creado correctamente:

```bash
docker images | grep contact-agenda-backend
```
![6-4-verify-backend-img.png](./images/6-4-verify-backend-img.png)

4. Se creara el Deployment para el backend. Crear el archivo `backend-deployment.yaml` con el siguiente contenido:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
   name: contact-backend
   namespace: contact-app
spec:
   replicas: 1
   selector:
      matchLabels:
         app: contact-backend
   template:
      metadata:
         labels:
            app: contact-backend
      spec:
         containers:
            - name: contact-backend
              image: contact-agenda-backend:latest
              imagePullPolicy: Never
              ports:
                 - containerPort: 8080
              env:
                 - name: MYSQL_PASSWORD
                   valueFrom:
                      secretKeyRef:
                         name: mysql-secret
                         key: mysql-root-password
```
![6-5-create-backend-deployment.png](./images/6-5-create-backend-deployment.png)
5. Crear en la carpeta `k8s` el archivo `backend-service.yaml` con el siguiente contenido para exponer el backend:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: contact-backend
  namespace: contact-app
spec:
  selector:
    app: contact-backend
  ports:
    - port: 8080
      targetPort: 8080
  type: ClusterIP
```
![6-6-create-backend-service.png](./images/6-6-create-backend-service.png)
6. Crear el Deployment del backend ejecutando el siguiente comando en la terminal:

```bash
kubectl apply -f k8s/backend-deployment.yaml
```
![6-7-create-backend-deployment-terminal.png](./images/6-7-create-backend-deployment-terminal.png)

7. Crear el Service del backend ejecutando el siguiente comando en la terminal:

```bash
kubectl apply -f k8s/backend-service.yaml
```
![6-8-create-backend-service-terminal.png](./images/6-8-create-backend-service-terminal.png)

8. Verificar que el Deployment del backend se haya creado correctamente:

```bash
kubectl get pods -n contact-app
```
![6-9-verify-backend-deployment.png](./images/6-9-verify-backend-deployment.png)

9. Verificar que el Service del backend se haya creado correctamente:

kubectl logs -f <pod-name> -n contact-app
```bash
kubectl logs -f contact-backend-6bfcbc9597-pjwxh -n contact-app
```
![6-10-verify-backend-service.png](./images/6-10-verify-backend-service.png)

10. Para acceder al backend desde el navegador, se debe hacer un port-forward del Service del backend. Ejecutar el siguiente comando en la terminal:

```bash
 kubectl port-forward svc/contact-backend 8080:8080 -n contact-app
```
![6-11-port-forward-backend.png](./images/6-11-port-forward-backend.png)

11. Abrir el navegador y acceder a la siguiente URL para verificar que el backend esté funcionando correctamente 'http://localhost:8080/swagger-ui/index.html'
![6-11-port-forward-backend-web.png](./images/6-11-port-forward-backend-web.png)


Paso 7: configurar Ingress para el backend

1. instalar el controlador de Ingress NGINX en Kubernetes. Ejecutar el siguiente comando en la terminal:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/cloud/deploy.yaml
```
![7-1-install-nginx-ingress.png](./images/7-1-install-nginx-ingress.png)
2. Verificar que el controlador de Ingress NGINX se haya instalado correctamente:

```bash
kubectl get pods -n ingress-nginx
```
![7-2-verify-nginx-ingress.png](./images/7-2-verify-nginx-ingress.png)
3. Crear el archivo `ingress.yaml` en la carpeta `k8s` con el siguiente contenido para configurar Ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: contact-ingress
  namespace: contact-app
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: contact.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: contact-backend
                port:
                  number: 8080
```
![7-3-create-ingress.png](./images/7-3-create-ingress.png)
4. Crear el Ingress ejecutando el siguiente comando en la terminal:

```bash
kubectl apply -f k8s/ingress.yaml
```
![7-4-create-ingress-terminal.png](./images/7-4-create-ingress-terminal.png)
5. Verificar que el Ingress se haya creado correctamente:

```bash
kubectl get ingress -n contact-app
```
![7-5-verify-ingress.png](./images/7-5-verify-ingress.png)
6. Para acceder al backend a través de Ingress, se debe agregar una entrada en el archivo `hosts` del sistema operativo. Abrir el archivo `/etc/hosts` (en Linux y macOS) o `C:\Windows\System32\drivers\etc\hosts` (en Windows) y agregar la siguiente línea:

```
sudo nano /etc/hosts
```
Se agrega esta línea al final del archivo:

```
127.0.0.1 contact.local
```
![7-6-host.png](./images/7-6-host.png)
8. Abrir el navegador y acceder a la siguiente URL para verificar que el backend esté funcionando correctamente a través de Ingress:

```
http://contact.local/swagger-ui/index.html
```
![7-7-verify-ingress-web.png](./images/7-7-verify-ingress-web.png)

# Paso 8: implementando Node Affinity 

1. Listar los nodos disponibles en el clúster de Kubernetes:

```bash
kubectl get nodes
```
![8-1-list-nodes.png](./images/8-1-list-nodes.png)

2. Definir una etiqueta personalizada para identificar el nodo como válido para contact-backend.
```bash
kubectl label nodes docker-desktop node-type=backend
```
Le pone la etiqueta node-type=backend a tu único nodo (docker-desktop), útil cuando tengas más de uno.
![8-2-label-node.png](./images/8-2-label-node.png)
3. Confirmar que la etiqueta se aplicó correctamente
```bash
kubectl get nodes --show-labels
```
![8-3-verify-label-node.png](./images/8-3-verify-label-node.png)
4. Crear un archivo `node-affinity.yaml` en la carpeta `k8s` con el siguiente contenido para implementar Node Affinity:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: contact-backend-affinity
  namespace: contact-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: contact-backend-affinity
  template:
    metadata:
      labels:
        app: contact-backend-affinity
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: node-type
                    operator: In
                    values:
                      - backend
      containers:
        - name: contact-backend
          image: contact-agenda-backend:latest
          imagePullPolicy: Never
          ports:
            - containerPort: 8080
          env:
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: mysql-root-password
```
![8-4-create-node-affinity.png](./images/8-4-create-node-affinity.png)
5. Crear el Deployment con Node Affinity ejecutando el siguiente comando en la terminal:

```bash
kubectl apply -f k8s/node-affinity.yaml
```
![8-5-create-node-affinity-terminal.png](./images/8-5-create-node-affinity-terminal.png)
6. Verificar que el Deployment con Node Affinity se haya creado correctamente:

```bash
kubectl get pods -n contact-app -o wide
```
![8-6-verify-node-affinity-t.png](./images/8-6-verify-node-affinity-t.png)
7. listar los pods para verificar que el pod contact-backend-affinity se esté ejecutando en el nodo con la etiqueta node-type=backend:

```bash
kubectl get pods -n contact-app
```

![8-7-verify-node-affinity-pod.png](./images/8-7-verify-node-affinity-pod.png)
8. Ver el log del pod con Node Affinity para verificar que se esté ejecutando correctamente:
```bash
 kubectl logs -f contact-backend-affinity-7d497cf4f8-69hvx -n contact-app
```
![8-8-verify-node-affinity-log.png](./images/8-8-verify-node-affinity-log.png)


## NOTA 
Al aplicar Node Affinity, le estamos diciendo a Kubernetes que este pod solo debe ejecutarse en nodos con ciertas características (en este caso, con la etiqueta node-type=backend). Aunque ahora solo tenemos un nodo, esta práctica es muy útil en entornos con múltiples nodos, ya que nos permite organizar mejor nuestras cargas de trabajo, aprovechar mejor los recursos y asegurar estabilidad.

# Paso 9: Aplicar monitoreo y Troubleshooting 

Es importante monitorear el estado de los pods y servicios en Kubernetes para asegurarse de que todo esté funcionando correctamente. Aquí hay algunos comandos útiles para monitorear y solucionar problemas:
 * Detectar errores antes de que impacten.
 * Ahorrar tiempo identificando la causa raíz de fallas.
 * Diagnosticar problemas de red, fallos de pods, o errores de configuración.

1. Instalar el componente metrics-server para monitorear el uso de recursos de los pods y nodos:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
![9-1-install-metrics-server.png](./images/9-1-install-metrics-server.png)
2. Verificar que el metrics-server se haya instalado correctamente:
```bash
kubectl get pods -n kube-system | grep metrics-server
```
3. Puede tardar un par de minutos en estar disponible, Si la salida es 0/1 seguir los siguientes pasos:
![9-2-verify-metrics-server.png](./images/9-2-verify-metrics-server.png)
4. Se Debe agregar --kubelet-insecure-tls  , para esto se debe modificar el Deployment del metrics-server. Para esto, ejecutar el siguiente comando en la terminal:
```bash
kubectl edit deployment metrics-server -n kube-system
```
![9-3-edit-metrics-server.png](./images/9-3-edit-metrics-server.png)
5. En la sección spec.template.spec.containers[0].args, agregá esta línea:

- --kubelet-insecure-tls
![9-4-add-kubelet-insecure-tls.png](./images/9-4-add-kubelet-insecure-tls.png)

6. Guardar y salir del editor. Esto actualizará el Deployment del metrics-server y aplicar los cambios del nuevo archivo
![9-5-save-metrics-server.png](./images/9-5-save-metrics-server.png)
7. Verificar nuevamente que el metrics-server esté funcionando correctamente:
```bash
kubectl get pods -n kube-system | grep metrics-server
```
![9-6-verify-metrics-server-running.png](./images/9-6-verify-metrics-server-running.png)
8. Verificar el uso de recursos de los pods en el namespace contact-app:
```bash
kubectl top nodes
kubectl top pods -n contact-app
```
![9-7-verify-metrics-server-top.png](./images/9-7-verify-metrics-server-top.png)

# Paso 10: Comandos clave de Troubleshooting

1. Verificar el estado de todos los recursos del namespac
```bash
kubectl get pods -n contact-app
```
![10-1-verify-pods-status.png](./images/10-1-verify-pods-status.png)
2. Ver eventos recientes del clúster como Fallos, reinicios , errores de sheduling, etc.
```bash
kubectl get events -n contact-app
```
![10-2-verify-events.png](./images/10-2-verify-events.png)
3. vamos a listar los pods para aplicar Troubleshooting
```bash
kubectl get pods -n contact-app
```
![10-3-list-pods.png](./images/10-3-list-pods.png)
4.  Ver detalle de un pod para ver eventos recientes como errores de inicio, reinicios o volumenes usamos el comando describe
```bash
kubectl describe pod contact-backend-6bfcbc9597-pjwxh -n contact-app
```
![10-4-describe-pod.png](./images/10-4-describe-pod.png)
5. Ver los logs de un pod para ver errores de la aplicación o problemas de inicio:
```bash
kubectl logs contact-backend-6bfcbc9597-pjwxh -n contact-app
```
![10-5-logs-pod.png](./images/10-5-logs-pod.png)
6. Para ingresar al pod con un shell interactivo y depurar problemas en tiempo real:
```bash
kubectl exec -it contact-backend-affinity-7d497cf4f8-69hvx -n contact-app -- /bin/sh
```
![10-6-exec-pod.png](./imges/10-6-exec-pod.png)
