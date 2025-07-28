# Práctica 1. Crear un contenedor PostgreSQL con Docker

## Objetivo de la práctica:

Al finalizar la práctica, serás capaz de:
- Estar familiarizado con las operaciones básicas de Docker, incluyendo la creación, gestión y manipulación de contenedores e imágenes.
- Aprender a ejecutar, detener, listar y reiniciar contenedores utilizando la interfaz de línea de comandos (CLI) de Docker, a fin de obtener las habilidades fundamentales para la gestión de contenedores y el despliegue de aplicaciones.

## Duración aproximada:
- 35 minutos.

---

**[Lista General](https://netec-mx.github.io/DOCK_KUB/)** | **[Siguiente ➡️](https://netec-mx.github.io/DOCK_KUB/Capitulo2/)**

---

## Instrucciones:

En esta práctica, asumirás el desafío de manejar contenedores con Docker, una herramienta clave en el desarrollo y despliegue de aplicaciones modernas. A continuación, sigue los pasos indicados para completar el ejercici.

### Tarea 1. Crear y ejecutar un contenedor Docker.

Paso 1. Accede a la consola para ejecutar comandos de CLI de Docker.

Paso 2. Crea y ejecuta un contenedor llamado `mi-contenedor` utilizando la imagen **httpd** con el siguiente comando. 

```bash 
docker run -d -p 8080:80 --name mi-contenedor httpd
```

![cap1_ejecucion_docker.png](../images/cap1_ejecucion_docker.png)

### Tarea 2. Uso de comandos. 

Para administrar Docker de manera eficiente, es fundamental conocer sus comandos básicos. A continuación, se presentan algunos de ellos para listar, detener, reiniciar y eliminar contenedores.

Paso 1. Con este comando, lista los contenedores en ejecución.

```bash
docker ps
```

![cap1_listar contenedores.png](../images/cap1_listarcontenedores.png)

Paso 2. Detén el contenedor `mi-contenedor` al ejecutar el comando siguiente.

```bash 
docker stop mi-contenedor
```

![cap1_detener_contenedor.png](../images/cap1_detener_contenedor.png)

Paso 3. Reinicia un contenedor con el comando:

```bash
docker restart mi-contenedor
```

![cap1_iniciar_contenedor.png](../images/cap1_iniciar_contenedor.png)

Paso 4. Elimina un contenedor al ejecutar el comando.

```bash 
docker rm mi-contenedor
```

![cap1_eliminar_contenedor.png](../images/cap1_eliminar_contenedor.png)

### Tarea 3. Descargar una imagen de Docker Hub; por ejemplo, Nginx.

Paso 1. Descarga la imagen Nginx desde Docker Hub ejecutando el siguiente comando:

```bash 
docker pull nginx:1.25
```

![cap1_nginx.png](../images/cap1_nginx.png)

Paso 2. Verifica y observa la imagen descargada al ejecutar:

```bash
docker images
```

![cap1_list_img.png](../images/cap1_list_img.png)

## Resultado esperado:

![cap1_reslt_final.png](../images/cap1_reslt_final.png)
