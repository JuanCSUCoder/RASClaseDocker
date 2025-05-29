## Parte 1: Introducción a Docker

[Volver al índice](/)

Docker es una plataforma para desarrollar, enviar y ejecutar aplicaciones usando contenedores. Los contenedores son unidades ligeras, independientes y ejecutables que empaquetan una aplicación y todas sus dependencias.

### Paso 1: Actualizar el sistema

Antes de instalar cualquier software nuevo, es una buena práctica asegurarse de que su sistema esté actualizado.

1.  Abra una terminal (Ctrl+Alt+T).
2.  Ejecute los siguientes comandos:

    ```bash
    sudo apt update
    sudo apt upgrade -y
    ```

### Paso 2: Instalar Docker Engine en Ubuntu 24.04 LTS

Vamos a instalar Docker Engine utilizando el repositorio oficial de Docker.

1.  **Instalar paquetes necesarios:**

    ```bash
    sudo apt install ca-certificates curl apt-transport-https software-properties-common -y
    ```

2.  **Agregar la clave GPG oficial de Docker:**

    ```bash
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg
    ```

3.  **Agregar el repositorio de Docker a APT sources:**

    ```bash
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

4.  **Actualizar el índice de paquetes e instalar Docker Engine:**

    ```bash
    sudo apt update
    sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
    ```

5.  **Verificar la instalación de Docker:**

    ```bash
    sudo systemctl status docker
    ```

    Debería ver un estado `active (running)`. Presione `q` para salir.

6.  **Ejecutar Docker sin `sudo` (Opcional pero recomendado para este taller):**

    Por defecto, para ejecutar comandos Docker, necesita usar `sudo`. Para evitar esto, puede agregar su usuario al grupo `docker`.

    ```bash
    sudo usermod -aG docker $USER
    ```

    Para que los cambios surtan efecto, debe cerrar la sesión y volver a iniciarla, o reiniciar su terminal. Después de esto, puede verificar que Docker funciona sin `sudo` ejecutando:

    ```bash
    docker run hello-world
    ```

    Debería ver un mensaje de "Hello from Docker!".

### Paso 3: Trabajando con Imágenes y Contenedores Docker

Una **imagen Docker** es una plantilla de solo lectura que contiene un conjunto de instrucciones para crear un contenedor. Un **contenedor Docker** es una instancia ejecutable de una imagen.

1.  **Descargar una imagen Docker:**

    ```bash
    docker pull ubuntu:22.04
    ```

    Esto descargará la imagen de Ubuntu 22.04.

2.  **Listar imágenes descargadas:**

    ```bash
    docker images
    ```

3.  **Ejecutar un contenedor desde una imagen:**

    ```bash
    docker run -it ubuntu:22.04 /bin/bash
    ```

    * `-it`: Asigna una pseudo-TTY y mantiene STDIN abierto, permitiéndote interactuar con el contenedor.
    * `ubuntu:22.04`: La imagen que queremos usar.
    * `/bin/bash`: El comando a ejecutar dentro del contenedor.

    ¡Ahora estás dentro del contenedor Ubuntu! Explora un poco (ej: `ls -l`, `pwd`). Para salir del contenedor, escribe `exit`.

4.  **Listar contenedores en ejecución:**

    ```bash
    docker ps
    ```

5.  **Listar todos los contenedores (incluidos los detenidos):**

    ```bash
    docker ps -a
    ```

6.  **Iniciar un contenedor detenido:**

    ```bash
    docker start <ID_DEL_CONTENEDOR_O_NOMBRE>
    ```

7.  **Detener un contenedor en ejecución:**

    ```bash
    docker stop <ID_DEL_CONTENEDOR_O_NOMBRE>
    ```

8.  **Eliminar un contenedor (debe estar detenido):**

    ```bash
    docker rm <ID_DEL_CONTENEDOR_O_NOMBRE>
    ```

9.  **Eliminar una imagen:**

    ```bash
    docker rmi <ID_DE_LA_IMAGEN_O_NOMBRE>
    ```
    *Asegúrate de no tener contenedores basados en esa imagen en ejecución o detenidos antes de intentar eliminarla.*

### Paso 4: Contenerizar una Aplicación Simple con Dockerfile

Vamos a crear una aplicación web muy simple (un "Hello World" con Python Flask) y la construiremos en una imagen Docker.

1.  **Crear un directorio para su proyecto:**

    ```bash
    mkdir my-flask-app
    cd my-flask-app
    ```

2.  **Crear el archivo `app.py`:**

    ```bash
    nano app.py
    ```

    Pegue el siguiente código Python:

    ```python
    from flask import Flask

    app = Flask(__name__)

    @app.route('/')
    def hello_world():
        return 'Hello from Docker!'

    if __name__ == '__main.main__':
        app.run(host='0.0.0.0', port=5000)
    ```

    Guarde y salga (Ctrl+X, Y, Enter).

3.  **Crear el archivo `requirements.txt`:**

    ```bash
    nano requirements.txt
    ```

    Pegue el siguiente contenido:

    ```
    flask
    ```

    Guarde y salga.

4.  **Crear el `Dockerfile`:**

    Un `Dockerfile` es un archivo de texto que contiene todas las instrucciones para construir una imagen Docker.

    ```bash
    nano Dockerfile
    ```

    Pegue el siguiente contenido:

    ```dockerfile
    # Usa una imagen base oficial de Python
    FROM python:3.9-slim-buster

    # Establece el directorio de trabajo dentro del contenedor
    WORKDIR /app

    # Copia los archivos de requerimientos y Python al contenedor
    COPY requirements.txt .
    COPY app.py .

    # Instala las dependencias
    RUN pip install --no-cache-dir -r requirements.txt

    # Expone el puerto que la aplicación escuchará
    EXPOSE 5000

    # Comando para ejecutar la aplicación cuando el contenedor se inicie
    CMD ["python", "app.py"]
    ```

    Guarde y salga.

5.  **Construir la imagen Docker:**

    Asegúrese de estar en el directorio `my-flask-app`. El `.` al final indica que el `Dockerfile` está en el directorio actual.

    ```bash
    docker build -t my-flask-app:1.0 .
    ```

    * `-t my-flask-app:1.0`: Asigna una etiqueta (nombre y versión) a su imagen.

    Esto tomará un tiempo mientras descarga la imagen base e instala las dependencias.

6.  **Verificar que la imagen se ha creado:**

    ```bash
    docker images
    ```

    Deberías ver `my-flask-app` en la lista.

7.  **Ejecutar el contenedor de su aplicación:**

    ```bash
    docker run -d -p 80:5000 --name flask-web-app my-flask-app:1.0
    ```

    * `-d`: Ejecuta el contenedor en modo "detached" (en segundo plano).
    * `-p 80:5000`: Mapea el puerto 80 de su máquina host al puerto 5000 del contenedor (donde su aplicación Flask está escuchando).
    * `--name flask-web-app`: Asigna un nombre al contenedor para facilitar su gestión.

8.  **Probar su aplicación:**

    Abra su navegador web y vaya a `http://localhost`. Debería ver "Hello from Docker!".

9.  **Detener y eliminar el contenedor:**

    ```bash
    docker stop flask-web-app
    docker rm flask-web-app
    ```

---