# Práctica 9. Trabajo con volúmenes emptyDir en Kubernetes

## Objetivo de la práctica:
Al finalizar la práctica, serás capaz de:
- Entender desde una manera práctica cómo interactuar con Kubernetes.
- Realizar operaciones básicas en Kubernetes.

## Duración aproximada:
- 80 minutos.

---

**[⬅️ Atrás](https://netec-mx.github.io/DOCK_KUB/Capitulo8/)** | **[Lista General](https://netec-mx.github.io/DOCK_KUB/)** | **[Siguiente ➡️](https://netec-mx.github.io/DOCK_KUB/Capitulo10/)**

---

## Instrucciones:
En esta práctica, comprenderás cómo utilizar **labels** y **selectors** para organizar y seleccionar recursos en Kubernetes.

### Tarea 1. Creación de un pod con labels y selectors.

Paso 1. Crea un yaml con el nombre de `pod1.yaml`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    app: myapp
    env: development
spec:
  containers:
    - name: nginx
      image: nginx
```

![cap9_labels_yaml.png](../images/cap9_labels_yaml.png)

Paso 2. Crea el pod con el siguiente comando:

```bash
kubectl apply -f pod1.yaml
```

![cap9_labels_yaml_create.png](../images/cap9_labels_yaml_create.png)

### Tarea 2. Listar los pods con labels y selectors.

Paso 1. Lista los pods con el siguiente comando:

```bash
kubectl get pods --show-labels
```

![cap9_list_all_labels_pod.png](../images/cap9_list_all_labels_pod.png)

Paso 2. Lista los pods con el selector `app=myapp`.

```bash
kubectl get pods -l app=myapp
```

![cap9_list_spesifict_label.png](../images/cap9_list_spesifict_label.png)

### Tarea 3. Actualizar los labels de un pod.

Paso 3. Actualiza los labels del **pod1** con el siguiente comando:

```bash
kubectl get pods -l app=myapp,env=development --show-labels
```

### Resultado esperado:

Al finalizar, podrás visualizar los labels de los pods.

![Screenshot 2025-03-22 at 2.35.13 PM.png](../images/cap9_resultado.png)
