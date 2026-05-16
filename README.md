# InnovaTech Chile - Etapa 2: Plataforma de Despachos y Ventas Contenerizada

Este repositorio contiene la arquitectura de microservicios, el frontend de administración y la automatización del pipeline de Integración y Despliegue Continuo (CI/CD) para la plataforma de **InnovaTech Chile**. La solución está completamente contenerizada utilizando **Docker** y se despliega de manera automatizada en una infraestructura multinivel sobre **AWS (Amazon Web Services)** mediante **GitHub Actions**.

---

## 🛠️ 1. Arquitectura de Contenerización

La solución implementa una separación estricta de responsabilidades (Frontend, Microservicios de Backend y Capa de Persistencia) mediante entornos aislados con Docker.

### Dockerfile (Buenas Prácticas y Multi-stage)
Cada componente del proyecto se construye utilizando estrategias de construcción multi-etapa (**Multi-stage Build**):

* **Frontend (`front_despacho`):** * *Etapa 1 (Build):* Utiliza una imagen base de `node:18-alpine` para instalar dependencias y compilar los archivos estáticos de React mediante Vite.
  * *Etapa 2 (Production):* Copia únicamente los artefactos compilados (`/dist`) hacia una imagen ultra-liviana de `nginx:alpine`. Esto minimiza el tamaño del contenedor final (pasando de ~1GB a <50MB) y mitiga vulnerabilidades al remover el entorno de ejecución de Node en producción.
* **Backends (`back-Despachos` y `back-Ventas`):**
  * Diseñados utilizando imágenes ligeras con soporte para OpenJDK de Java, optimizando las capas de caché de Docker al separar el código fuente de las dependencias pesadas `.jar`.
  * Configurados bajo el principio de **Mínimo Privilegio**, asegurando que los servicios no corran bajo usuario root siempre que sea posible.

### Configuración del Servidor Web (Nginx como Proxy Reverso)
El frontend actúa como el único punto de entrada perimetral de la aplicación. Nginx está configurado para enrutar el tráfico dinámico y estático de forma segura:
* Las solicitudes tradicionales al ecosistema web (`/`) cargan el SPA de React de forma nativa.
* Las peticiones bajo el contexto `/api/v1/despachos` y `/api/v1/ventas` son interceptadas y redirigidas transparentemente mediante `proxy_pass` hacia las IPs privadas internas correspondientes de los microservicios sin exponer los puertos de backend al internet público.

---

## 💾 2. Estrategia de Persistencia de Datos

Para garantizar la continuidad operativa de la empresa y asegurar que la información crítica no se pierda ante el reinicio, falla o actualización de un contenedor, se implementa persistencia de datos en el motor de bases de datos.

* **Tecnología:** Named Volumes (Volúmenes Nominados de Docker).
* **Configuración:** `db_data:/var/lib/mysql`
* **Justificación Técnica:** Se seleccionaron **Named Volumes** por encima de los *Bind Mounts* debido a que los volúmenes con nombre son gestionados directamente por el ciclo de vida del motor de Docker. Esto garantiza:
  1. Un rendimiento superior de entrada/salida (I/O) nativo de Linux en AWS EC2 para bases de datos transaccionales como MySQL.
  2. Un aislamiento de seguridad estricto frente al sistema de archivos del Host.
  3. Desacoplamiento total: las imágenes del backend o la base de datos pueden ser destruidas y actualizadas por el pipeline de CI/CD sin comprometer la integridad de las tablas existentes.

---

## 🚀 3. Pipeline de Integración y Despliegue Continuo (CI/CD)

La automatización de entregas está gestionada mediante un workflow nativo en **GitHub Actions** configurado en `.github/workflows/main_deploy.yml`.

### Flujo del Pipeline (Build ➡️ Push ➡️ Deploy)
El pipeline se activa automáticamente ante eventos de `push` dirigidos estrictamente a la rama **`deploy`** y ejecuta los siguientes jobs correlativos:
[ Push a rama 'deploy' ]
│
▼

Job: Build & Push ──────────────────► Compila imágenes locales (Frontend y Backends)
│                                Autentica en Docker Hub de forma segura
│                                Publica tags ':latest' actualizados
▼

Job: Deploy (AWS) ──────────────────► Configura credenciales temporales de AWS
Invoca AWS Systems Manager (SSM)
Ejecuta comandos de Docker Run en caliente (EC2)


1. **Build & Push:** Levanta un entorno virtual limpio, descarga el código fuente, inicia sesión en Docker Hub y construye en paralelo las imágenes del Frontend y de ambos microservicios, empujándolas al registro público del perfil correspondiente.
2. **Despliegue en AWS (SSM):** En lugar de exponer llaves SSH de producción comprometedoras, el pipeline se comunica con la nube configurando credenciales AWS y utiliza **AWS Systems Manager (AWS-RunShellScript)** para inyectar comandos de terminal de forma interna y encriptada directo en las instancias EC2 del Frontend, Backend y Base de Datos.

### Gestión de Variables de Entorno y GitHub Secrets
Toda la información sensible, credenciales y direccionamiento IP se encuentra resguardada en los **Repository Secrets** de GitHub, previniendo fugas de credenciales en texto plano dentro del código:
* `DOCKERHUB_USERNAME` / `DOCKERHUB_PASSWORD`
* `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` / `AWS_SESSION_TOKEN`
* `EC2_FRONTEND_INSTANCE_ID` / `EC2_BACKEND_INSTANCE_ID` / `EC2_DB_INSTANCE_ID`
* `IP_PRIVADA_DB`

---

## 🔒 4. Arquitectura de Red y Seguridad en AWS

El despliegue respeta estrictamente el diseño de red de AWS enfocado en la seguridad y las políticas de acceso restringido:

* **Instancia Frontend (Subred Pública):** Expone únicamente el puerto `80` (HTTP) de cara a internet. Es la única máquina con una IP pública accesible desde el navegador del cliente.
* **Instancias de Backend y Base de Datos (Subredes Privadas):** No poseen acceso IP público directo desde internet.
  * El Backend procesa peticiones en los puertos `8081` y `8082` únicamente provenientes del rango de la VPC o la IP privada asignada al Frontend.
  * Los microservicios de Spring Boot consumen la base de datos inyectando en su entorno las variables requeridas de conexión (`DB_ENDPOINT`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`), impidiendo fallas de inicialización y garantizando el aislamiento total.
  * La base de datos MySQL bloquea el puerto `3306` para todo tráfico externo mediante reglas estrictas de **Security Groups**, aceptando exclusivamente conexiones legítimas del backend.

---

## 👥 Integrantes
* Joaquín Fuenzalida Leal
* Elías León

**Profesor:** Eric Ramirez  
**Institución:** DUOC UC - Introducción a Herramientas DevOps
