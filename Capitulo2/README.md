# Práctica 2. Gestión avanzada de contenedores Docker con Node.js

## Objetivo de la práctica:
Al finalizar la práctica, serás capaz de:
- Profundizar en aspectos avanzados del manejo de Docker, enfocándose en el manejo de logs, la interacción avanzada con contenedores, la limitación de recursos, el monitoreo y la manipulación de imágenes Docker.
- Desarrollar habilidades críticas para el mantenimiento eficiente y la gestión de aplicaciones en contenedores, aprendiendo a diagnosticar y resolver problemas comunes, optimizar el rendimiento y asegurar la eficiencia de los recursos.

## Duración aproximada:
- 45 minutos.

---

**[⬅️ Atrás](https://netec-mx.github.io/DOCK_KUB/Capitulo1/)** | **[Lista General](https://netec-mx.github.io/DOCK_KUB/)** | **[Siguiente ➡️](https://netec-mx.github.io/DOCK_KUB/Capitulo3/)**

---

## Instrucciones:
En esta práctica te enfrentarás al desafío de aplicar técnicas avanzadas en la gestión de contenedores Docker.

### Tarea 1. Configuración de una Aplicación NodeJS.

Paso 1. Crea una carpeta llamada `seconLab`.

![cap2_create_file_project.png](../images/cap2_create_file_project.png)

Paso 2. Crea un proyecto backend en **Node.js** con un endpoint y, posteriormente, dockerízalo. Para ello, ubícate en la carpeta `secondLab` y ejecuta el siguiente comando:

```bash
npm init -y|
```

![cap2_node_json.png](../images/cao2_node_json.png)

Paso 3. Una vez ejecutado el comando, se generará el archivo **package.json**.

![cap2_json_file.png](../images/cap2_json_file.png)

Paso 4. Crea un archivo llamado `app.js` con el siguiente contenido:

```javascript
const express = require('express');
const app = express();
const port = 3000;
app.get('/', (req, res) => {
    res.send('Hola Docker!');
});
app.listen(port, () => {
    console.log(`Aplicación escuchando en http://localhost:${port}`);
});
```
![cap2_app_js_file.png](../images/cap2_app_js_file.png)

Paso 5. Instala **Express** ejecutando el siguiente comando:

```bash
npm install express --save
```

![cap2_install_express.png](../images/cap2_install_express.png)

Paso 6. Ejecuta la aplicación con el siguiente comando:

```bash
node app.js
```

![cap2_start_express.png](../images/cap2_start_express.png)

Paso 7. Abre tu navegador y verifica que la aplicación esté funcionando correctamente.

![cap2_start_express_web.png](../images/cap2_start_express_web.png)

### Tarea 2. Dockerfile para la Aplicación NodeJS.

Paso 1. Crea un archivo llamado `Dockerfile` en la carpeta `seconLab` con el siguiente contenido:

```dockerfile
FROM node:latest
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "app.js"]
```

![cap2_dockerfile.png](../images/cap2_dockerfile.png)

Paso 2. Construye la imagen Docker con el siguiente comando:

```bash
docker build -t lab2 .
```

![cap2_build_image.png](../images/cap2_build_image.png)

Paso 3. Ejecuta el contenedor con el siguiente comando:

```bash
docker run -d -p 3000:3000 lab2
```

![cap2_run_container.png](../images/cap2_run_container.png)

Paso 4. Verifica que la aplicación esté funcionando correctamente en el navegador.

![cap2_start_express_web.png](../images/cap2_start_express_web.png)

### Tarea 3. Logs de un Contenedor.

Paso 1. Consulta los logs del contenedor con el siguiente comando:

```bash
docker logs <container_id>
```

![cap2_logs_container.png](../images/cap2_logs_container.png)

Paso 2. Verifica los logs del contenedor en tiempo real con el siguiente comando:

```bash
docker logs -f <container_id>
```

![cap2_logs_container_follow.png](../images/cap2_logs_container_follow.png)


### Resultado esperado:

Al finalizar, habrás adquirido habilidades avanzadas en el manejo de contenedores Docker, aplicando técnicas avanzadas en la gestión de logs, la interacción avanzada con contenedores, la limitación de recursos, el monitoreo y la manipulación de imágenes Docker.

![cap2_start_express_web.png](../images/cap2_start_express_web.png)
