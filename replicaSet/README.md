[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/josephefranco)

# Introduccion a los ReplicaSet

Resumen

- [x] Correr nuestro primer pod
- [x] Creacion del primer pod por yaml
- [x] Varios contenedores en un mismo Pod 
- [x] Algunas cosas más
- [ ] Revision de esta guia (esto se escribio de forma rapida, mientras aprendia la plataforma con lo cual pueden existir correcciones o mejoras. No dudes de hacerme un PR para ir mejorando el contenido)


## ¿Qué es un ReplicaSet?
ReplicaSet es un recurso de Kubernetes que asegura que siempre se ejecute un número de réplicas de un pod determinado. Por lo tanto, nos asegura que un conjunto de pods siempre están funcionando y disponibles. Nos proporciona las siguientes características:

* Que no haya caída del servicio
* Tolerancia a errores
* Escalabilidad dinámica

![Alt text](../asserts/rs01.png?raw=true "Title")



## Creacion de un ReplicaSet

Tenemos un ejemplo simple en [rs.yaml](rs.yaml), dentro de los valores que indicamos es la cantidad de replicas que necesitamos del pod. Es importante que el selector/matchLabels haga match con los label de nuestro pod y no tengamos otros pods que los comparta sino va a interactuar con ellos. Con esto vemos la importante de tener definido un buen estandar con los labels de nuestras aplicaciones para no tener problemas a futuro.

Aplicamos el nuevo manifiesto, en el mismo especificamos que necesitamos tiene 3.

```
$ kubectl apply -f rs.yaml 
replicaset.apps/rs-test created
```

Si buscamos en los pods, vemos que se sumaron 3 mas.. de nuestra replica.

```
$ kubectl get pods
NAME                  READY   STATUS        RESTARTS   AGE
my-pod                1/1     Running       0          154m
my-pod-twocontainer   2/2     Running       0          119m
my-pod2               1/1     Running       0          152m
my-pod3               1/1     Running       0          152m
my-pod4               1/1     Running       0          144m
my-pod5               1/1     Running       0          128m
nginx                 1/1     Running       0          109m
nginx2                1/1     Running       0          109m
podtest2              1/1     Running       0          122m
podtest3              1/1     Running       0          122m
podtest4              1/1     Running       0          122m
rs-test-5b2s9         1/1     Running       0          17s
rs-test-jcx6n         1/1     Running       0          17s
rs-test-tsp9h         1/1     Running       0          17s
```

Pero kubectl nos permite visualizar el estado del replicaSet
```
$ kubectl get rs
NAME      DESIRED   CURRENT   READY   AGE
rs-test   3         3         3       2m
```

Entonces, por lo que podemos analizar los pods se generaron con el prefijo del nombre de nuestro replicaSet que especificamos.
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-test
```

Entonces ahora tiene mas coherente cuando buscamos nuestros pods.
```
$ kubectl get pods | grep rs-test
rs-test-5b2s9         1/1     Running   0          2m48s
rs-test-jcx6n         1/1     Running   0          2m48s
rs-test-tsp9h         1/1     Running   0          2m48s
```

Tambien podemos revisar nuestro rs para ver que pods fueron creados
```
$ kubectl describe rs/rs-test

  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  9m38s  replicaset-controller  Created pod: rs-test-jcx6n
  Normal  SuccessfulCreate  9m38s  replicaset-controller  Created pod: rs-test-tsp9h
  Normal  SuccessfulCreate  9m38s  replicaset-controller  Created pod: rs-test-5b2s9
```

Si consultamos la definicion de alguno de estos pods
```
$ kubectl get pods rs-test-jcx6n -o yaml
```

Vamos a ver que dentro del yaml especifica la referencia al owner, que seria el replicaSet.
``` 
ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: rs-test
    uid: 169f61d5-3083-4239-9537-06920665b01c
```

En resumen, los parametros que tenemos son: 
* replicas: Indicamos el número de pos que siempre se van a estar ejecutando.
* selector: Indicamos el pods que vamos a replicar y vamos a controlar con el ReplicaSet. En este caso va a controlar pods que tenga un label app cuyo valor sea nginx. Si no se indica el campo selector se seleccionar or defecto los pods cuyos labels sean iguales a los que hemos declarado en la sección siguiente.
* template: El recurso ReplicaSet contiene la definición de un pod.


## Creacion de otro ReplicaSet

Creamos dos pods que tenemos definido en el [pod-rs.yaml](pod-rs.yaml) 
```
$ kubectl apply -f pod-rs.yaml
pod/pod1 created
pod/pod2 created
```

Verificamos y vemos que tenemos dos pods corriendo, los filtro por el label de tier que se definicio en el manifiesto.
```
$ kubectl get pods -l tier=frontend
NAME             READY   STATUS    RESTARTS   AGE
pod1             1/1     Running   0          4m55s
pod2             1/1     Running   0          4m55s
```

Creamos un replicaSet que necesita 3 replicas

```
$ kubectl apply -f rs rs-frrontend.yaml
replicaset.apps/frontend created
```

Verificamos el estado de nuestro rs

```
kubectl get rs frontend
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       17m
```

Pero al verificar nuestros pods, vemos lo siguiente

```
$ kubectl get pods -l tier=frontend
NAME             READY   STATUS    RESTARTS   AGE
frontend-8tzjb   1/1     Running   0          17m
pod1             1/1     Running   0          18m
pod2             1/1     Running   0          18m

```
Que paso? Como en el manifiesto especifica en el selector que teniamos tier=frontend , confundio los otros dos pods que teniamos en la replica

Si borramos los dos pods creados anteriores.

```
$ kubectl delete -f pods-rs.yaml 
pod "pod1" deleted
pod "pod2" deleted
```

Revisamos nuevamente los pods y ahora si aparecen los pods de nuestro replicaSet.
```
$ kubectl get pods -l tier=frontend
NAME             READY   STATUS    RESTARTS   AGE
frontend-8tzjb   1/1     Running   0          21m
frontend-j87j6   1/1     Running   0          25s
frontend-xvw2r   1/1     Running   0          25s
```

Con esto entendemos lo importante que son los labels y los selectos.


## ¿Es lo mismo un ReplicaSet de un Replication Controller?
 
Los conjuntos de réplica son una especie de híbrido, ya que en algunos aspectos son más potentes que los controladores de replicación, y en otros son menos potentes. Los conjuntos de réplica se declaran esencialmente de la misma manera que los controladores de réplica, excepto que tienen más opciones para el selector. 

Tenemos para hacer la comparativa el yaml que se genero en  la prueba del replication controller y su equivalente como replica sets.
Por favor revisar el [rc.yaml](../replicationController/rc.yaml) y [rs.yaml](../replicationController/rs.yaml).

En este caso ([rs.yaml](../replicationController/rs.yaml)), es más o menos lo mismo que cuando creamos el Controlador de replicación, excepto que estamos usando matchLabels en lugar de etiqueta. Pero podríamos haber dicho con la misma facilidad:

```
spec:
   replicas: 3
   selector:
     matchExpressions:
      - {key: app, operator: In, values: [my-custom-app-rct, my-custom-app-rct-release]}
      - {key: teir, operator: NotIn, values: [production]}
  template:
     metadata:
```

En este caso, estamos viendo dos condiciones diferentes:

* La etiqueta de la aplicación debe ser my-custom-app-rct o my-custom-app-rct-release
* La etiqueta de nivel (si existe) no debe ser producción


## Algunas cosas más,


+ TBD



## ¿Es lo mismo un ReplicaSet de un Replication Controller?
 
Los conjuntos de réplica son una especie de híbrido, ya que en algunos aspectos son más potentes que los controladores de replicación, y en otros son menos potentes. Los conjuntos de réplica se declaran esencialmente de la misma manera que los controladores de réplica, excepto que tienen más opciones para el selector. Por ejemplo, podríamos crear un conjunto de réplicas como este:


En este caso, es más o menos lo mismo que cuando creamos el Controlador de replicación, excepto que estamos usando matchLabels en lugar de etiqueta. Pero podríamos haber dicho con la misma facilidad:


...
spec:
   replicas: 3
   selector:
     matchExpressions:
      - {key: app, operator: In, values: [soaktestrs, soaktestrs, soaktest]}
      - {key: teir, operator: NotIn, values: [production]}
  template:
     metadata:
...


En este caso, estamos viendo dos condiciones diferentes:
La etiqueta de la aplicación debe ser soaktestrc, soaktestrs o soaktest
La etiqueta de nivel (si existe) no debe ser producción.


Pueden mas informacion en https://kubernetes.io/es/docs/concepts/overview/working-with-objects/common-labels/


## Alternativas de ReplicaSet
 
### Deployment (recomendado)
La implementación es un objeto que puede poseer ReplicaSets y actualizarlos, así como sus Pods de forma declarativa, en el lado del servidor y con actualizaciones continuas.

Si bien los ReplicaSets se pueden usar de forma independiente, Deployments los usa principalmente hoy como mecanismo para organizar la creación, eliminación y actualización de Pods. Cuando use Implementaciones, ya no tendrá que preocuparse por administrar los ReplicaSets así creados. Las implementaciones poseen y administran sus ReplicaSets. Es por eso que se recomienda usar implementaciones cuando desee ReplicaSets. En el proximo capitulo vamos a profundizar sobre este tema.

Pueden revisar el apartado anterior sobre [Deployments](../deployment/README.md)


### Pods
A diferencia de cuando un usuario creó pods directamente, un ReplicaSet reemplaza los pods eliminados o terminados por cualquier motivo, por ejemplo, en caso de falla de un nodo o mantenimiento de nodo disruptivo, como una actualización. día del grano Por esta razón, le recomendamos que use un ReplicaSet incluso si su aplicación requiere solo un pod. Piénselo de la misma manera que un supervisor de procesos, pero él supervisa múltiples pods en múltiples nodos en lugar de procesos individuales en un solo nodo. Un ReplicaSet delega reinicios de contenedores locales a un agente en el nodo (por ejemplo, Kubelet o Docker).

Pueden revisar el apartado anterior sobre [Pods](../pods/README.md)

### Job
Use un trabajo en lugar de un ReplicaSet para los pods que deben terminar por sí mismos (es decir, trabajos por lotes).

### DaemonSet
Utilice un DaemonSet en lugar de un ReplicaSet para los pods que proporcionan funcionalidad a nivel de nodo, como la supervisión o el registro de ese nodo. Estas cápsulas tienen una vida útil que está vinculada a la vida útil de una máquina: la cápsula debe estar funcionando en la máquina antes del inicio de las otras cápsulas y seguramente terminará cuando la máquina esté lista para reiniciarse. / detenido

### ReplicationController
Los ReplicaSets son los sucesores de ReplicationControllers. Ambos tienen el mismo propósito y se comportan de la misma manera, excepto que ReplicationController no es compatible con los requisitos del selector descritos en las etiquetas de la guía del usuario. Como tal, los ReplicaSets son preferibles a los ReplicationControllers.