### Paso 5: Limpieza

[Volver al índice](/)

Es importante detener y eliminar los recursos cuando haya terminado para liberar los recursos del sistema.

1.  **Eliminar el Service y el Deployment:**

    ```bash
    kubectl delete -f flask-service.yaml
    kubectl delete -f flask-deployment.yaml
    ```

2.  **Restaurar el entorno de Docker a su configuración predeterminada:**

    ```bash
    eval $(minikube docker-env -u)
    ```

    Esto deshace los cambios realizados por `eval $(minikube docker-env)`.

3.  **Detener y eliminar el clúster de Minikube:**

    ```bash
    minikube stop
    minikube delete
    ```

    Esto eliminará la máquina virtual de Minikube y todos sus datos.

---

## Conclusión

¡Felicidades! Ha completado un taller introductorio a Docker y Kubernetes. Ha aprendido a:

* Instalar Docker y ejecutar contenedores.
* Contenerizar una aplicación simple con un Dockerfile.
* Configurar Minikube para un clúster de Kubernetes local.
* Desplegar una aplicación en Kubernetes utilizando Deployments y Services.

Esto es solo el comienzo de su viaje en el mundo de los contenedores y la orquestación. ¡Explore más a fondo los vastos recursos disponibles para profundizar su conocimiento en Docker y Kubernetes!

**Próximos pasos sugeridos:**

* Investigar sobre Docker Compose para orquestar múltiples contenedores Docker.
* Explorar más a fondo los diferentes tipos de Services en Kubernetes (ClusterIP, LoadBalancer, Ingress).
* Aprender sobre volúmenes persistentes en Kubernetes para datos.
* Probar a escalar su aplicación en Kubernetes modificando el número de réplicas en su Deployment.
* Considerar el uso de un registro de contenedores (como Docker Hub) para almacenar y compartir sus imágenes.

¡Gracias por participar en este taller!