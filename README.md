## Orquestación con Docker Compose

Este directorio contiene el archivo `docker-compose.yml` necesario para orquestar las tres aplicaciones del proyecto Lista de Tareas Todo Seguro:

- **mongodb**: base de datos MongoDB endurecida.
- **api**: API Node.js/Express segura.
- **frontend**: aplicación Angular servida con Nginx endurecido.

Todas las imágenes hacen referencia a builds ya subidas al registry `docker.io/focasoftware` con configuraciones de seguridad (usuarios no-root, `no-new-privileges`, file system de solo lectura donde aplica, `tmpfs`, etc.).

---

## ✅ Requisitos previos

- **Docker**  instalado y funcionando.
- **Docker Compose** v2 (o `docker compose` integrado en Docker Desktop).
- Conexión a internet para poder descargar las imágenes desde Docker Hub.

---

## ⚙️ Variables de entorno

El `docker-compose.yml` utiliza variables de entorno para no hardcodear credenciales ni URLs.
Antes de levantar los servicios, crea un archivo `.env` en este mismo directorio con el siguiente contenido de ejemplo:

```env
# MongoDB
MONGO_INITDB_ROOT_USERNAME=username
MONGO_INITDB_ROOT_PASSWORD=password
MONGO_INITDB_DATABASE=dbname

# API
NODE_ENV=env-production-develop
PORT=port
JWT_SECRET=SecretoSuperLargoYUnico

# URL
FRONTEND_URL=http://localhost:80
```

> **IMPORTANTE:** No subas el archivo `.env` a repositorios públicos. Contiene credenciales y secretos.

---

## 🚀 Levantar las aplicaciones

1. **Situarte en el directorio:**

```bash
cd entregable-3
```

2. **Crear el archivo `.env`** (si aún no existe), usando como base el ejemplo anterior y ajustando los valores a tu caso.

3. **Levantar los servicios en segundo plano:**

```bash
docker compose up -d
```

Esto descargará y levantará los 3 contenedores:

- `mongodb-secure`
- `api-secure`
- `frontend-secure`

4. **Ver el estado de los contenedores:**

```bash
docker compose ps
```

5. **Ver logs en tiempo real :**

```bash
docker compose logs -f
```

---

## 🌐 Acceso a la aplicación

Con los contenedores en ejecución:

- **Frontend**  
  - URL: `http://localhost:8888`

- **API**  
  - Puerto expuesto: `http://localhost:3000`
  - Ejemplo de endpoint de salud:  
    `http://localhost:3000/health`

- **MongoDB**  
  - Host dentro de la red de Docker: `mongodb`
  - Puerto mapeado en el host: `27017`
  - URI de conexión desde fuera de Docker (por ejemplo, un cliente Mongo GUI):

```text
mongodb://MONGO_INITDB_ROOT_USERNAME:MONGO_INITDB_ROOT_PASSWORD@localhost:27017/MONGO_INITDB_DATABASE?authSource=admin
```

Sustituye las variables por los valores que configuraste en tu `.env`.


Una vez que los contenedores están levantados con `docker compose up -d` y accedes a `http://localhost:8888`, la aplicación se ve así:

![Aplicación en ejecución](img/todoseguro.png)

---

## ⛔ Detener y limpiar contenedores

Para **detener** los servicios:

```bash
docker compose down
```

Para detenerlos y además **eliminar el volumen de datos de MongoDB** (¡perderás los datos!):

```bash
docker compose down -v
```

---

## 🔐 Seguridad aplicada

Algunos puntos relevantes del `docker-compose.yml`:

- **Red dedicada** `app-network` para aislar los servicios.
- **Volumen persistente** `mongodb_data` para los datos de MongoDB.
- `security_opt: no-new-privileges:true` en los tres servicios para evitar escalación de privilegios.
- **Sistemas de archivos de solo lectura** (`read_only: true`) para `api` y `frontend`.
- Uso de **`tmpfs`** para directorios temporales y logs, reduciendo la superficie de ataque y escritura en disco.
- Las imágenes se referencian usando **digest (`@sha256:...`)** para asegurar inmutabilidad y evitar sorpresas por cambios en tags.
