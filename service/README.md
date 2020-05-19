[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/josephefranco)

# Introduccion a los Deployments

Resumen

- [ ] TBD
- [ ] Revision de esta guia (esto se escribio de forma rapida, mientras aprendia la plataforma con lo cual pueden existir correcciones o mejoras. No dudes de hacerme un PR para ir mejorando el contenido)


## ¿Qué es un Servicio?

Una forma abstracta de exponer una aplicación que se ejecuta en un conjunto de Pods como un servicio de red.

Con Kubernetes, no necesita modificar su aplicación para utilizar un mecanismo de descubrimiento de servicios desconocido. Kubernetes proporciona a los Pods sus propias direcciones IP y un solo nombre DNS para un conjunto de Pods, y puede equilibrar la carga entre ellos.


Los contenedores de Kubernetes son mortales. Nacen y cuando mueren, no resucitan. Si usa una Implementación para ejecutar su aplicación, puede crear y destruir Pods dinámicamente.

Cada Pod obtiene su propia dirección IP, sin embargo, en una Implementación, el conjunto de Pods que se ejecutan en un momento podría ser diferente del conjunto de Pods que ejecutan esa aplicación un momento después.

Esto lleva a un problema: si algún conjunto de Pods (llámelos "backends") proporciona funcionalidad a otros Pods (llámelos "frontends") dentro de su clúster, ¿cómo se enteran los front-end y realizan un seguimiento de a qué dirección IP conectarse? , para que el frontend pueda usar la parte de backend de la carga de trabajo?




## ¿Por qué usar un servicio?
En un clúster de Kubernetes, cada pod tiene una dirección IP interna. Sin embargo, los pods van y vienen en una implementación, y sus direcciones IP cambian. Por este motivo, no tiene sentido usar direcciones IP de pod directamente. Con un servicio, obtienes una dirección IP estable que dura toda la vida del servicio, incluso cuando las direcciones IP de los pods miembros cambian.

Además, un servicio proporciona el balanceo de cargas. Los clientes llaman a una única dirección IP estable, y sus solicitudes se balancean en los pods que son miembros del servicio.

## Tipos de servicios
Existen cinco tipos de servicios:

* ClusterIP (predeterminado): los clientes internos envían solicitudes a una dirección IP interna estable. Solo se tiene visibilidad dentro del nodo.

* NodePort: los clientes envían solicitudes a la dirección IP de un nodo en uno o más valores nodePort que el servicio especifica. Pueden acceder externamente al nodo.

* LoadBalancer: los clientes envían solicitudes a la dirección IP de un balanceador de cargas de red. Se necesita para usarlo en AWS, Azure, GCP, etc.

* ExternalName: los clientes internos usan el nombre de DNS de un servicio como un alias para un nombre de DNS externo.

* Headless: puedes usar un servicio sin interfaz gráfica cuando quieras tener una agrupación de pods, pero no necesites contar con una dirección IP estable.

El tipo NodePort es una extensión del tipo ClusterIP. Entonces, un servicio de tipo NodePort tiene una dirección IP de clúster.

El tipo LoadBalancer es una extensión del tipo NodePort. Entonces, un servicio de tipo LoadBalancer tiene una dirección IP de clúster y uno o más valores nodePort.



```
$ kubectl apply -f svc.yaml
service/my-service created
```
 


 clusterip es una ip virtual del clusterip

```
$ kubectl get svc

NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    9h
my-service   ClusterIP   10.110.37.238   <none>        8080/TCP   11s
```
 
```
$ kubectl get endpoints

NAME         ENDPOINTS                                            AGE
kubernetes   192.168.61.24:8443                                   9h
my-service   172.17.0.17:8080,172.17.0.21:8080,172.17.0.22:8080   6m41s 
```
 
 
```
$ kubectl describe svc my-service

Name:              my-service
Namespace:         default
Labels:            app=backend
                   lab=my-first-service
Annotations:       Selector:  app=backend,lab=my-first-service
Type:              ClusterIP
IP:                10.97.99.53
Port:              <unset>  8080/TCP
TargetPort:        8080/TCP
Endpoints:         172.17.0.17:8080,172.17.0.21:8080,172.17.0.22:8080
Session Affinity:  None
Events:            <none>
```



```
$ kubectl get pods

NAME                               READY   STATUS    RESTARTS   AGE
deployment-test-598db649b6-64hhl   1/1     Running   0          49s
deployment-test-598db649b6-j8c64   1/1     Running   0          49s
deployment-test-598db649b6-x8cgx   1/1     Running   0          49s
my-pod                             1/1     Running   0          8h
my-pod-twocontainer                2/2     Running   0          7h38m
my-pod-with-env-vars               1/1     Running   0          4h3m
my-pod-with-ref-vars               1/1     Running   0          3h51m
my-pod2                            1/1     Running   0          8h
my-pod3                            1/1     Running   0          8h
my-pod4                            1/1     Running   0          8h
my-pod5                            1/1     Running   0          7h47m
nginx                              1/1     Running   0          7h27m
nginx2                             1/1     Running   0          7h27m
podtest2                           1/1     Running   0          7h41m
podtest3                           1/1     Running   0          7h41m
podtest4                           1/1     Running   0          7h41m
rs-test-5b2s9                      1/1     Running   0          5h38m
rs-test-jcx6n                      1/1     Running   0          5h38m
rs-test-tsp9h                      1/1     Running   0          5h38m
```


```
$ kubectl run  -it --rm terminal-temp  --image=praqma/network-multitool  -- sh

If you don't see a command prompt, try pressing enter.
/ # curl http://10.97.99.53:8080
Estoy corriendo en el nodo minikube, mi nombre es deployment-test-598db649b6-x8cgx y me dieron la ip 172.17.0.21
/ # curl http://10.97.99.53:8080
Estoy corriendo en el nodo minikube, mi nombre es deployment-test-598db649b6-x8cgx y me dieron la ip 172.17.0.21
/ # curl http://10.97.99.53:8080
Estoy corriendo en el nodo minikube, mi nombre es deployment-test-598db649b6-j8c64 y me dieron la ip 172.17.0.17
/ # curl http://10.97.99.53:8080
Estoy corriendo en el nodo minikube, mi nombre es deployment-test-598db649b6-64hhl y me dieron la ip 172.17.0.22
/ # curl http://10.97.99.53:8080
```



```
$ kubectl get endpoints my-service -o yaml | grep -e "name\|ip"

  name: my-service
  namespace: default
  selfLink: /api/v1/namespaces/default/endpoints/my-service
  - ip: 172.17.0.17
      name: deployment-test-598db649b6-j8c64
      namespace: default
  - ip: 172.17.0.21
      name: deployment-test-598db649b6-x8cgx
      namespace: default
  - ip: 172.17.0.22
      name: deployment-test-598db649b6-64hhl
      namespace: default
```



## ClusterIP


## NodePort
Como su propio nombre indica, NodePort es el acceso a través de un puerto de los nodos, que son las máquinas que forman tu clúster. Vamos a verlo con un ejemplo:

Hoy vamos a utilizar la imagen aspnetapp que está en la cuenta de Microsoft de Docker Hub. Creamos un pod con dos réplicas que escuchan por el puerto 80.

kubectl run aspnetapp \
--image=mcr.microsoft.com/dotnet/core/samples:aspnetapp \
--port=80 \
--replicas=2
Una vez que el despliegue se está ejecutando correctamente, (puedes comprobarlo a través de kubectl get deployments) vamos a exponer la aplicación a través de un servicio del tipo NodePort:

kubectl expose deployment aspnetapp --type=NodePort
Cuando exponemos un servicio utilizando este tipo, nuestro clúster nos asignará un puerto entre el rango 30000–32767. Para saber qué puerto nos ha sido asignado en los nodos, podemos lanzar el comando kubectl get services.


En mi caso estoy trabajando con las IPs 192.168.1.48 (master), 192.168.1.49 (nodo1) y 192.168.1.51 (nodo2) para mis nodos. Si ahora accedo a través del navegador a cualquiera de ellos, especificando el puerto 30207 accederé a mi aplicación aspnetapp.