# Guía de Despliegue de Aplicaciones en Contenedores Docker

![Mapa conceptual Docker](./docs/mapaConceptualDocker.png)


Este repositorio contiene la arquitectura de microservicios y la configuración necesaria para empaquetar, publicar y desplegar una aplicación multicapa utilizando Docker, Docker Compose y flujos de integración continua (CI/CD).

El proyecto está diseñado como un estándar técnico y de aprendizaje para la gestión moderna de infraestructura de software y despliegue en contenedores.

---

## 1. ¿Qué hace la solución?

La solución permite empaquetar y aislar la aplicación junto con sus dependencias en contenedores ligeros y portables, garantizando la consistencia del entorno tanto en desarrollo como en producción.

### Funcionalidades principales

* **Aislamiento de entornos:** Encapsulamiento de la aplicación (frontend/backend) y la base de datos en entornos independientes pero interconectados.
* **Orquestación simplificada:** Gestión de múltiples servicios mediante `docker-compose.yml` para levantar la infraestructura completa con una sola instrucción.
* **Automatización CI/CD:** Publicación automática de imágenes de Docker hacia un registro de contenedores (Docker Hub o GitHub Packages) tras cada confirmación en la rama principal.
* **Persistencia y Redes:** Configuración de volúmenes para el almacenamiento persistente de datos y redes virtuales dedicadas para asegurar la comunicación interna entre servicios.

---

## 2. Requisitos de Instalación por Sistema Operativo

Antes de ejecutar la aplicación, asegúrese de cumplir con los siguientes requisitos en su sistema operativo:

### Windows
* Windows 10/11 Home, Pro, Enterprise o Education (64 bits).
* WSL 2 (Windows Subsystem for Linux 2) habilitado.
* Virtualización habilitada en el BIOS/UEFI.
* Git instalado en la línea de comandos.

### macOS
* macOS versión 10.15 (Catalina) o posterior.
* Arquitectura basada en procesadores Intel o Apple Silicon (M1/M2/M3).
* Git instalado (`xcode-select --install`).

### Linux (Ubuntu / Debian / RHEL)
* Kernel de Linux versión 3.10 o superior.
* Acceso a la terminal con privilegios de superusuario (`sudo`).
* Git instalado (`sudo apt install git` o equivalente según la distribución).

---

## 3. Instalación Paso a Paso

Siga las instrucciones específicas para su sistema operativo hasta levantar la solución completa con `docker compose up -d`.

---

### Opción A: Instalación en Windows

#### Paso 1: Habilitar WSL 2
1. Preparar el motor de WSL. Abra PowerShell como Administrador (clic derecho sobre el menú Inicio  → Terminal (Administrador)). Antes de pensar en Ubuntu, deje instalado y actualizado el motor del subsistema:
```
winver                           # la version de Windows debe ser 2004 o superior
wsl --version                    # si responde "parametro no valido", su WSL es antiguo
wsl --install --no-distribution  # instala/actualiza SOLO el motor, sin distribucion
wsl --update                     # trae la version de la Store, con el catalogo moderno
wsl --shutdown
wsl --set-default-version 2
```
***La clave está en --no-distribution: instala el motor moderno de WSL sin intentar descargar todavía ninguna distribución, que es justo el punto donde se rompe el comando de un solo paso.***
Si `wsl -version` respondió que el parámetro no es válido, su equipo traía la versión antigua del subsistema; después de estos comandos ya tendrá la actual.

2. Confirmar el nombre exacto de la distribución, antes de instalarla. 
Todavía en PowerShell como Administrador:
`wsl --list --online`
***Verifique: en la columna NAME debe aparecer Ubuntu-24.04. Si el paso 1 se ejecutó completo, el nombre ya está en la lista.***
No pase al paso siguiente sin haberlo visto.

3. Instalar Ubuntu. En la misma ventana de PowerShell como Administrador:
`wsl --install -d Ubuntu-24.04 --web-download`
El modificador `--web-download` descarga la distribución desde los servidores de Microsoft en lugar de la Microsoft Store. Es necesario en equipos de sala o corporativos, donde la Store suele estar deshabilitada por política del equipo, o no hay una cuenta Microsoft iniciada.
***Reinicie el equipo si el sistema se lo solicita.***

4. Primer arranque de Ubuntu. 
Este paso se hace una sola vez, y es donde más se traba quien nunca ha usado una terminal.
 - Espere la ventana negra. Al terminar la descarga se abre sola una ventana con el texto 
**Installing, this may take a few minutes....** 
Puede tardar varios minutos. No la cierre. Si por accidente la cierra, abra Ubuntu desde el menú Inicio y el proceso continúa donde iba.
 - Cree el usuario de Linux. Cuando la instalación termina, aparece el texto 
`Enter new UNIX username:`. Escriba un nombre en minúsculas, sin espacios, sin tildes y sin ñ y pulse Enter. No tiene que ser el mismo usuario con el que entra a Windows.
 - Cree la contraseña. Aparece `New password:`. Escriba una contraseña y pulse Enter aunque en la pantalla no se vea absolutamente nada.
 - Repítala. Aparece `Retype new password:`. Escriba exactamente la misma y pulse Enter. Si responde `Sorry, passwords do not match`, simplemente vuelve a pedirla desde el principio.

5. Confirmar que la distribución quedó en WSL 2. De vuelta en PowerShell:
`wsl --list --verbose`
Verifique: la línea de Ubuntu-24.04 debe mostrar VERSION 2. Si muestra VERSION 1, conviértala con `wsl -set-version Ubuntu-24.04 2` y espere a que termine. Sobre WSL 1 no hay núcleo Linux real y Docker Engine no funcionará.

6. Habilitar systemd, que es el que administrará el servicio de Docker. Abra Ubuntu desde el menú Inicio y ejecute:
```sudo tee /etc/wsl.conf > /dev/null <<EOF
[boot]
systemd=true
EOF
```
Cierre la ventana de Ubuntu y, desde PowerShell, reinicie el subsistema:
`wsl --shutdown`
Vuelva a abrir Ubuntu desde el menú Inicio y compruebe que systemd está activo:
`systemctl is-system-running`   # debe responder running o degraded, no offline
*Reinicie el sistema si la consola se lo solicita.*

#### Paso 2: Instalar Docker Engine

1. Retirar paquetes no oficiales que puedan entrar en conflicto
```
sudo apt remove -y docker.io docker-compose docker-compose-v2 docker-doc podman-docker
```
2. Registrar el repositorio oficial de Docker
```
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
sudo tee /etc/apt/sources.list.d/docker.sources > /dev/null <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```
3. Instalar el motor, la CLI y los plugins de Buildx y compose
```
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```
4. Arrancar el servicio y dejarlo habilitado al inicio del sistema
```
sudo systemctl enable --now docker
```
5. Poder usar docker sin sudo
```
sudo usermod -aG docker $USER
```

#### Paso 3: Clonar el repositorio y ejecutar la solución
Abra Git Bash, PowerShell o CMD y ejecute:

```powershell
# Clonar el repositorio
git clone https://github.com/christopherFall/app-deploy-docker.git

# Ingresar al directorio del proyecto
cd app-deploy-docker

# Construir y levantar los contenedores en segundo plano
docker compose up -d
```

---

### Opción B: Instalación en macOS

#### Paso 1: Instalar Docker Engine con Colima

 - Instalar Homebrew, si aun no lo tiene
```
/bin/bash -c "$(curl -fsSL \
  https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
 - Instalar el cliente de Docker, el plugin de Compose y Colima
`brew install docker docker-compose colima`

 - Preparar la carpeta de configuracion del cliente
`mkdir -p ~/.docker`

 - Crear y arrancar la maquina virtual con Docker Engine dentro
`colima start --cpu 2 --memory 4 --disk 20`

 - Comprobar el estado de la maquina virtual
`colima status`
Falta registrar el plugin de Compose ante el cliente de Docker. Si el archivo `~/.docker/config.json` no existe, créelo con este contenido; si ya existe, agréguele únicamente la clave 
```
cliPluginsExtraDirs:
{
  "cliPluginsExtraDirs": ["/opt/homebrew/lib/docker/cli-plugins"]
}
```
Esa es la ruta en Mac con Apple Silicon. En Mac con procesador Intel la ruta es `/usr/local/lib/docker/cli-plugins`. 
Si tiene dudas, ejecute `brew --prefix` y use el valor que devuelva seguido de `/lib/docker/cli-plugins`.

#### Paso 2: Clonar el repositorio y ejecutar la solución
Abra la Terminal de macOS y ejecute:

```bash
# Clonar el repositorio
git clone https://github.com/christopherFall/app-deploy-docker.git

# Ingresar al directorio del proyecto
cd app-deploy-docker

# Construir y levantar los contenedores en segundo plano
docker compose up -d
```

---

### Opción C: Instalación en Linux (Ubuntu/Debian)

#### Paso 1: Instalar Docker Engine y Docker Compose
Ejecute la siguiente secuencia de comandos en su terminal:

```bash
# Actualizar el índice de paquetes e instalar dependencias básicas
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release

# Agregar la clave GPG oficial de Docker
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Configurar el repositorio estable de Docker
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Instalar Docker Engine, CLI, Containerd y el plugin de Docker Compose
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Agregar el usuario actual al grupo docker para evitar el uso de sudo en cada comando
sudo usermod -aG docker $USER
newgrp docker
```

#### Paso 2: Clonar el repositorio y ejecutar la solución
```bash
# Clonar el repositorio
git clone https://github.com/christopherFall/app-deploy-docker.git

# Ingresar al directorio del proyecto
cd app-deploy-docker

# Construir y levantar los contenedores en segundo plano
docker compose up -d
```

---

### Verificación del despliegue (Todos los Sistemas Operativos)

Para verificar que los servicios estén en ejecución:

```bash
# Consultar el estado de los contenedores activos
docker compose ps

# Inspeccionar los registros de ejecución
docker compose logs -f
```

Para detener la infraestructura:

```bash
docker compose down
```

---

## 4. Publicación Automática de la Imagen (CI/CD)

El proyecto utiliza **GitHub Actions** para automatizar el proceso de construcción (*build*) y publicación (*push*) de las imágenes de Docker hacia un registro de contenedores (Docker Hub).

### Flujo de Trabajo (Workflow)

Cada vez que se realiza un evento `push` o un `pull_request` a la rama principal (`main` o `master`), se activa de forma automática la pipeline definida en el archivo `.github/workflows/deploy.yml`.

### Estructura típica del archivo de automatización (`.github/workflows/deploy.yml`)

```yaml
name: Publicación Automática en Docker Hub

on:
  push:
    branches:
      - main
      - master

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Descargar el código fuente
        uses: actions/checkout@v4

      - name: Iniciar sesión en Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Configurar Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Construir y publicar la imagen
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: |
            ${{ secrets.DOCKERHUB_USERNAME }}/app-deploy-docker:latest
            ${{ secrets.DOCKERHUB_USERNAME }}/app-deploy-docker:${{ github.sha }}
```

### Requisitos previos en GitHub

Para que la publicación automática funcione correctamente, se deben configurar los Secretos del Repositorio (*Repository Secrets*):

1. Ir al repositorio en GitHub -> **Settings** -> **Secrets and variables** -> **Actions**.
2. Crear la variable secret: `DOCKERHUB_USERNAME` con el nombre de usuario de Docker Hub.
3. Crear la variable secret: `DOCKERHUB_TOKEN` con un Access Token generado desde Docker Hub (en **Account Settings** -> **Security** -> **New Access Token**).

### Resultado del Proceso
1. El código se valida y empaqueta en un entorno aislado de GitHub.
2. La imagen compilada se etiqueta con `:latest` y con el hash único del commit.
3. La imagen queda disponible de manera pública o privada en Docker Hub para ser desplegada en cualquier servidor remoto.