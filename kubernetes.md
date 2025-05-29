## Parte 2: Introducción a Kubernetes con Minikube

[Volver al índice](/)

Kubernetes es un sistema de orquestación de contenedores de código abierto que automatiza el despliegue, escalado y gestión de aplicaciones contenerizadas. Para este taller, usaremos Minikube, una herramienta que le permite ejecutar un clúster de Kubernetes de un solo nodo en su máquina local.

### Paso 1: Instalar Minikube y Kubectl

**Kubectl** es la herramienta de línea de comandos para interactuar con clústeres de Kubernetes.

1.  **Instalar Kubectl:**

    ```bash
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    rm kubectl
    ```

2.  **Verificar la instalación de Kubectl:**

    ```bash
    kubectl version --client
    ```

3.  **Instalar Minikube:**

    ```bash
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    sudo install minikube-linux-amd64 /usr/local/bin/minikube
    rm minikube-linux-amd64
    ```

4.  **Verificar la instalación de Minikube:**

    ```bash
    minikube version
    ```

### Paso 2: Iniciar un Clúster de Minikube

Minikube necesita un controlador (driver) para crear la máquina virtual donde se ejecutará Kubernetes. El driver por defecto es `docker` si ya lo tienes instalado y configurado correctamente, lo cual hicimos.

1.  **Iniciar Minikube:**

    ```bash
    minikube start --driver=docker
    ```

    Esto tomará unos minutos mientras descarga las imágenes de Kubernetes e inicializa el clúster. Si encuentra errores, asegúrese de que Docker esté en ejecución y que su usuario esté en el grupo `docker`.

2.  **Verificar el estado del clúster:**

    ```bash
    minikube status
    ```

    Debería ver algo como `host: Running`, `kubelet: Running`, `apiserver: Running`, `kubeconfig: Configured`.

3.  **Verificar los nodos del clúster (con Kubectl):**

    ```bash
    kubectl get nodes
    ```

    Debería ver un nodo llamado `minikube` con el estado `Ready`.

### Paso 3: Desplegar una Aplicación en Kubernetes

Ahora desplegaremos nuestra aplicación Flask contenerizada en nuestro clúster de Minikube. En Kubernetes, las unidades más pequeñas desplegables son los **Pods**. Los **Deployments** gestionan los Pods, asegurando que un número específico de réplicas estén siempre en ejecución. Los **Services** exponen las aplicaciones que se ejecutan en un conjunto de Pods a la red.

1.  **Indicar a Docker que use el demonio de Minikube (IMPORTANTE):**

    Para que Minikube pueda usar las imágenes que usted ha construido localmente, necesita indicarle a su cliente Docker que hable con el demonio de Docker dentro de Minikube.

    ```bash
    eval $(minikube docker-env)
    ```

    Después de ejecutar este comando, cualquier comando `docker build` o `docker images` que ejecute se referirá al entorno de Docker dentro de Minikube.

2.  **Reconstruir la imagen de su aplicación (dentro del entorno de Minikube):**

    Asegúrese de estar en el directorio `my-flask-app`.

    ```bash
    docker build -t my-flask-app:1.0 .
    ```

    Esto construirá la imagen directamente dentro del entorno de Docker de Minikube, lo que significa que Kubernetes podrá encontrarla sin tener que "pullarla" de un registro externo.

3.  **Crear un Deployment para su aplicación:**

    Un Deployment describe cómo desplegar su aplicación.

    ```bash
    nano flask-deployment.yaml
    ```

    Pegue el siguiente YAML:

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: flask-app-deployment
    spec:
      replicas: 1 # Puedes cambiar esto para tener más instancias
      selector:
        matchLabels:
          app: flask-app
      template:
        metadata:
          labels:
            app: flask-app
        spec:
          containers:
          - name: flask-app-container
            image: my-flask-app:1.0 # Usamos nuestra imagen local
            imagePullPolicy: Never # Indicamos a Kubernetes que no intente descargar la imagen de un registro
            ports:
            - containerPort: 5000
    ```

    Guarde y salga.

4.  **Aplicar el Deployment:**

    ```bash
    kubectl apply -f flask-deployment.yaml
    ```

    Debería ver `deployment.apps/flask-app-deployment created`.

5.  **Verificar el Deployment y los Pods:**

    ```bash
    kubectl get deployments
    kubectl get pods
    ```

    Debería ver el `flask-app-deployment` y un Pod asociado con un estado `Running`.

### Paso 4: Exponer la Aplicación con un Service

Un Service en Kubernetes es una abstracción que define un conjunto lógico de Pods y una política para acceder a ellos.

1.  **Crear un Service para su aplicación:**

    ```bash
    nano flask-service.yaml
    ```

    Pegue el siguiente YAML:

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: flask-app-service
    spec:
      selector:
        app: flask-app # Coincide con las etiquetas de los Pods del Deployment
      ports:
        - protocol: TCP
          port: 80 # Puerto en el Service
          targetPort: 5000 # Puerto en el contenedor
      type: NodePort # Expone el Service en un puerto en los nodos del clúster
    ```

    Guarde y salga.

2.  **Aplicar el Service:**

    ```bash
    kubectl apply -f flask-service.yaml
    ```

    Debería ver `service/flask-app-service created`.

3.  **Verificar el Service:**

    ```bash
    kubectl get services
    ```

    Debería ver `flask-app-service` con un `TYPE` de `NodePort`.

4.  **Acceder a su aplicación:**

    Minikube proporciona un comando conveniente para abrir su navegador al Service expuesto.

    ```bash
    minikube service flask-app-service
    ```

    Esto abrirá su navegador web predeterminado en la URL de su aplicación en ejecución. ¡Debería ver "Hello from Docker!" de nuevo, esta vez entregado por Kubernetes!
