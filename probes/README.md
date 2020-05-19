[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/josephefranco)

# Introduccion a los Deployments

Resumen

- [ ] TBD
- [ ] Revision de esta guia (esto se escribio de forma rapida, mientras aprendia la plataforma con lo cual pueden existir correcciones o mejoras. No dudes de hacerme un PR para ir mejorando el contenido)
 
## Introduccion
Kubelet hace el probe, y un intervalo de tiempo hace un control y toma una accion.


El kubelet utiliza sondas de preparación para saber cuándo un contenedor está listo para comenzar a aceptar tráfico. Una pod se considera listo cuando todos sus contenedores están listos. Un uso de esta señal es controlar qué Pods se usan como backends para los Servicios. Cuando un Pod no está listo, se elimina de los equilibradores de carga de servicio.

Leer https://www.innoq.com/en/blog/kubernetes-probes/


## Probes disponibles

Tenemos distintos tipos de checkeos: 

* Liveness: esto se ejecuta en un intervalo de tiempo, un ejemplo seria verificar una pagina web recurrentemente, si en algun momento se vuelve inestable ya sea por problema de codigo o un factor externo, y nos retorna un error 500 podriamos fallar el liveness y el pod destruirse, para dar inicio a uno nuevo sin este defecto.
* Readiness: es un diagnostico que se hace de ponerlo disponible el pod, una vez que pasa el readliness el pod queda disponible en el servicio.  
* Startup: Si el statup esta definido, se pausan los anteriores. Este se usa para aplicaciones muy pesadas, o que necesitan muchas rutinas de iniciacion.



## Que tipo de Probes soporta?

* Liveness command: ejecuta un comando en el contenedor, si el resultado es distinto a 0 fallo el mismo.
* Liveness HTTP request: hace una peticion a un puerto y si esta abierto es satisfactorio.
* TCP liveness probe: Hace una peticion http, mientras que el http code no sea de error da satisfactorio esta prueba.

##  Liveness command
Muchas aplicaciones que se ejecutan durante largos períodos de tiempo eventualmente pasan a estados rotos y no pueden recuperarse excepto al reiniciarse. Kubernetes proporciona sondas de vitalidad para detectar y remediar tales situaciones.

liviness-cmd.yaml


Este ejemplo verifica que el archivo /tmp/healthy, el control hace cada 5segundos. En el Pod despues de un intervalo lo borra y por lo consiguiente se destruye y se vuelve a contruir el pod.

```
$ kubectl apply -f liviness-cmd.yaml

pod/liveness-exec created
```

```
$ kubectl get pods

NAME            READY   STATUS    RESTARTS   AGE
liveness-exec   1/1     Running   0          6s
```

```
$ kubectl get pods   

NAME            READY   STATUS    RESTARTS   AGE
liveness-exec   1/1     Running   3          4m10s
```

```
$ kubectl describe pod liveness-exec
  Normal   Started    2m12s (x3 over 4m40s)  kubelet, minikube  Started container liveness
  Warning  Unhealthy  89s (x9 over 4m9s)     kubelet, minikube  Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
  Normal   Killing    89s (x3 over 3m59s)    kubelet, minikube  Container liveness failed liveness probe, will be restarted
  Normal   Pulling    59s (x4 over 4m44s)    kubelet, minikube  Pulling image "k8s.gcr.io/busybox"  
```


## TCP liveness probe
A third type of liveness probe uses a TCP socket. With this configuration, the kubelet will attempt to open a socket to your container on the specified port. If it can establish a connection, the container is considered healthy, if it can’t it is considered a failure.

Ver el archivo liviness-tcp.yaml.

El Liveness verifica cada x tiempo y destruye el pod, Readiness noticica que el pod no puede recibir mas carga y no le llega mas trafico.
 
 
## Liveness HTTP request
Another kind of liveness probe uses an HTTP GET request. Here is the configuration file for a Pod that runs a container based on the k8s.gcr.io/liveness image.


va a probar livenessProbe cada 15 segundos para ver que siga funcionando mediante un get al /healthz


Ver el archivo liviness-cmd.yaml.