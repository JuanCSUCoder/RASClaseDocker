### Paso 3: Desplegar una Aplicación en Kubernetes

Ahora desplegaremos nuestra aplicación Flask contenerizada en su clúster de pruebas de Kubernetes. En Kubernetes, las unidades más pequeñas desplegables son los **Pods**. Los **Deployments** gestionan los Pods, asegurando que un número específico de réplicas estén siempre en ejecución. Los **Services** exponen las aplicaciones que se ejecutan en un conjunto de Pods a la red.

1.  **Crear un Deployment para su aplicación:**

    Un Deployment describe cómo desplegar su aplicación.

    ```bash
    vode flask-deployment.yaml
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
            image: your_dockerhub_username/my-flask-app:1.0 # ¡IMPORTANTE! Reemplace con la ruta completa de su imagen en el registro
            ports:
            - containerPort: 5000
    ```

    Guarde y salga.

2.  **Aplicar el Deployment:**

    ```bash
    kubectl apply -f flask-deployment.yaml
    ```

    Debería ver `deployment.apps/flask-app-deployment created`.

3.  **Verificar el Deployment y los Pods:**

    ```bash
    kubectl get deployments
    kubectl get pods
    ```

    Debería ver el `flask-app-deployment` y un Pod asociado con un estado `Running`. Si el Pod no se inicia, verifique los logs del Pod con `kubectl logs <nombre_del_pod>` y `kubectl describe pod <nombre_del_pod>`.

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
          port: 80 # Puerto en el nodo
          targetPort: 5000 # Puerto en el contenedor
      type: NodePort
    ```

    **Nota:** Para un clúster de pruebas en la nube, `LoadBalancer` es el tipo de servicio más común para exponer externamente. Si su clúster de pruebas es muy básico o local (sin un balanceador de carga integrado), podría necesitar cambiar `type: LoadBalancer` a `type: NodePort`.

    Guarde y salga.

2.  **Aplicar el Service:**

    ```bash
    kubectl apply -f flask-service.yaml
    ```

    Debería ver `service/flask-app-service created`.

3.  **Verificar el Service y obtener la IP externa:**

    ```bash
    kubectl get services
    ```

    Busque su `flask-app-service`. Si usó `type: LoadBalancer`, espere unos momentos a que se le asigne una `EXTERNAL-IP`. Esta IP será la dirección a la que debe acceder desde su navegador. Si usó `NodePort`, verá un puerto en el formato `30000:80/TCP` bajo la columna `PORT(S)`.

    **Si usó `NodePort`:** Necesitará la IP pública de uno de los nodos de su clúster y el puerto `NodePort` asignado. Puede obtener la IP de los nodos con `kubectl get nodes -o wide`.

4.  **Acceder a su aplicación:**

    Abra su navegador web y vaya a la **EXTERNAL-IP** (si usó LoadBalancer) o `http://<IP_del_nodo>:<NodePort_asignado>` (si usó NodePort). Debería ver "Hello from Docker!" entregado por su aplicación en Kubernetes.
