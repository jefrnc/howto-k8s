[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/josephefranco)

# Introduccion a los Deployments

Resumen

- [ ] TBD
- [ ] Revision de esta guia (esto se escribio de forma rapida, mientras aprendia la plataforma con lo cual pueden existir correcciones o mejoras. No dudes de hacerme un PR para ir mejorando el contenido)
- [ ] Metric Server?
 
## Introduccion

Los Request y los Limits son los mecanismos que Kubernetes usa para controlar recursos como la CPU y la memoria. Las solicitudes son lo que el contenedor tiene garantizado. Si un contenedor solicita un recurso, Kubernetes solo lo programará en un nodo que pueda darle ese recurso. Los límites, por otro lado, aseguran que un contenedor nunca supere un cierto valor. El contenedor solo puede subir hasta el límite, y luego está restringido.
Es importante recordar que el límite nunca puede ser inferior a la solicitud. Si intenta esto, Kubernetes arrojará un error y no le permitirá ejecutar el contenedor.

Las solicitudes y los límites son por contenedor. Si bien los pods generalmente contienen un solo contenedor, también es común ver las pods con múltiples contenedores. Cada contenedor en el Pod obtiene su propio límite y solicitud individual, pero debido a que los Pods siempre se programan como un grupo, debe agregar los límites y las solicitudes de cada contenedor para obtener un valor agregado para el Pod.


https://cloud.google.com/blog/products/gcp/kubernetes-best-practices-resource-requests-and-limits
https://sysdig.com/blog/kubernetes-limits-requests/

## ¿Qué es un Request?
Las solicitudes son lo que el contenedor tiene garantizado. Por ejemplo, si un contenedor solicita un recurso, Kubernetes solo lo programará en un nodo que pueda darle ese recurso.

## ¿Qué es un Limit?
Los límites, por otro lado, son el umbral de recursos que un contenedor nunca excede. El contenedor solo puede subir hasta el límite, y luego está restringido.
 
## El ciclo de vida de una Pod de Kubernetes
Es importante comprender cómo los pods obtienen recursos y qué sucede cuando exceden lo que se asigna o la capacidad general de su clúster para que pueda ajustar sus contenedores y la capacidad del clúster correctamente. A continuación se muestra el flujo de trabajo de programación de pod típico:

1. Usted asigna recursos a su pod a través de la especificación y ejecuta el comando kubectl apply
2. El planificador de Kubernetes (scheduker) usará la política de programación de turnos para elegir un Nodo para ejecutar su pod
    * Para cada nodo, Kubernetes comprueba si el nodo tiene suficientes recursos para cumplir con las solicitudes de recursos.
3. El pod está programado para que el primer nodo tenga suficientes recursos.
4. El pod pasa al estado pendiente si ninguno de los nodos disponibles tiene suficientes recursos.
    * Si está utilizando algun escalador automático(AKS,CA, etc), su clúster escalará el número de nodos para asignar más capacidad.
5. Si un nodo está ejecutando pods donde la suma de sus límites supera su capacidad disponible, Kubernetes entra en estado de sobrecompromiso (Overcommitted State).
    * Debido a que la CPU se puede comprimir, Kubernetes se asegurará de que sus contenedores obtengan la CPU que solicitaron y acelerará el resto.
    * La memoria no se puede comprimir, por lo que Kubernetes necesita comenzar a tomar decisiones sobre qué contenedores terminar si el Nodo se queda sin memoria.


## ¿Qué pasa si Kubernetes entra en un estado comprometido?
Tenga en cuenta que Kuberentes optimiza el estado y la disponibilidad de todo el sistema. Cuando entra en el estado de sobrecompromiso, el planificador de Kubernetes puede tomar la decisión de matar los pods. En general, si un pod está utilizando recursos más de lo solicitado, ese pod se convierte en candidato para la terminación. Aquí tienes las condiciones que debes tener en cuenta:

* Un pod no tiene una solicitud de CPU y memoria definida. Kubernetes considera que este pod usa más de lo que solicitó por defecto.
* Un pod tiene una solicitud definida pero está utilizando más de lo solicitado, pero por debajo de su límite.
* Si varios pods superan sus solicitudes, Kubernetes las clasificará según su prioridad. Los pods de menor prioridad se terminan primero.
* Si todos los pods tienen la misma prioridad, Kubernetes comienza con el pod que más está revisando los recursos solicitados.
* Kubernetes puede matar los pods dentro de su solicitud si un componente crítico, como el motor Kubelet o Docker, está utilizando más recursos de los que están reservados para ellos. Estos son casos raros.




## Limitar RAM en el Pod

Ejecutamos la prueba limit-ram.yaml. En este se solicita 100Mi (esto esta garantizado), el tope podria ser 200Mi (siempre y cuando tenga disponibilidad el nodo, pero no esta garantizado.) El proceso va a consumir 150M, con lo cual esta dentro de las posibilidades aunque no esta garantizado.


Aplicamos el manifiesto

```
kubectl apply -f limit-ram.yaml
pod/memory-demo created
```
 
Verificamos si funciona, y esta todo ok.
```
$ kubectl get pods | grep memory-demo
memory-demo                         1/1     Running            0          29s
```

Si ahora ejecutamos un caso similar, con un request de 100Mi,  un limit de 200Mi. Pero con un proceso que va a ocupar 250Mi, veamos que pasa.
El archivo que vamos a usar es limit-ram2.yaml
 
```
$ kubectl apply -f limit-ram2.yaml
pod/memory-demo2 created4
```

Verificamos el estado
```
$ kubectl get pods | grep memory-demo2
memory-demo2                        0/1     OOMKilled          0          5s
```
Cuando el POD tiene un "límite" de memoria (máximo) definido y si el uso de la memoria del POD cruza más allá del límite especificado, el POD se eliminará y el estado se informará como OOMKilled. Tenga en cuenta que esto sucede a pesar de que el nodo tiene suficiente memoria libre.


Con el tiempo vemos que no pude levantar
```
$ ubectl get pods | grep memory-demo2
memory-demo2                        0/1     CrashLoopBackOff   4          2m39s
```


Ahora vamos a probar, si un pod solicita mas memoria de la disponible en el pod.
```
$ kubectl apply -f limit-ram3.yaml
pod/memory-demo3 created

```

Verificamos el estado del pod
```
$ kubectl get pods | grep memory-demo3
memory-demo3                        0/1     Pending            0          7s
```

Va a quedar Peding, hasta encontrar un nodo que pueda satisfacer sus necesidades.

```
$ kubectl describe pod memory-demo3

Events:
  Type     Reason            Age        From               Message
  ----     ------            ----       ----               -------
  Warning  FailedScheduling  <unknown>  default-scheduler  0/1 nodes are available: 1 Insufficient memory.
  Warning  FailedScheduling  <unknown>  default-scheduler  0/1 nodes are available: 1 Insufficient memory.

```


## Limitar Recursos de CPU


Limitamos el consumo de 1 CPU/100m, Request 0.5/500m y vamos a tener un procesamiento de 2CPU.

Ejecutamos el caso de limit-cpu.yaml
```
$ kubectl apply -f limit-cpu.yaml  
pod/cpu-demo created
```

```
$ kubectl get pods cpu-demo

NAME       READY   STATUS    RESTARTS   AGE
cpu-demo   1/1     Running   0          14s
```

A diferencia de la limitacion de Memoria, en el CPU limita el procesamiento y no mata el pod. 

Si quiero ver el estado actual del nodo puedo hacer lo siguiente
```
$ kubectl describe node minikube
 Resource           Requests     Limits
  --------           --------     ------
  cpu                1250m (62%)  1 (50%)
  memory             140Mi (3%)   340Mi (8%)
  ephemeral-storage  0 (0%)       0 (0%)
  hugepages-2Mi      0 (0%)       0 (0%)
```
 
Ahora vayamos al caso de pedir mas CPU de que tengamos disponibles el caso de limit-cpu2.yaml

```
$ kubectl apply -f limit-cpu2.yaml 
pod/cpu-demo2 created

```
```
$ kubectl get pods cpu-demo2      

NAME        READY   STATUS    RESTARTS   AGE
cpu-demo2   0/1     Pending   0          9s
```

```
$ skubectl describe pod cpu-demo2
Events:
  Type     Reason            Age        From               Message
  ----     ------            ----       ----               -------
  Warning  FailedScheduling  <unknown>  default-scheduler  0/1 nodes are available: 1 Insufficient cpu.
  Warning  FailedScheduling  <unknown>  default-scheduler  0/1 nodes are available: 1 Insufficient cpu.
```

En conclusion: 

* Solicitud (request): la cantidad mínima del recurso que el planificador reserva para que la utilice el contenedor. Si la cantidad es igual al límite, la solicitud está garantizada. Si la cantidad es menor que el límite, la solicitud sigue estando garantizada, pero el planificador puede utilizar la diferencia entre la solicitud y el límite para asignarla a recursos de otros contenedores.

* Límite (limit): la cantidad máxima del recurso que puede consumir el contenedor. Si la cantidad total de recursos que se utiliza en los contenedores supera la cantidad disponible en el nodo trabajador, los contenedores pueden ser desalojados para liberar espacio. Para evitar el desalojo, defina una solicitud de recurso igual al límite del contenedor. Si no se especifica ningún límite, el valor predeterminado es la capacidad del nodo trabajador.

Mas adelante vamos a ver que podrian haber definido quotas a nivel de namespace con lo cual podriamos no tener recursos disponibles, para ver que disponibilidad tenemos deberiamos ejecutar lo siguiente:
```
$kubectl get quota --namespace=prod
```


## QoS classes
Sin duda, es uno de los conceptos que se menciona cuando empezamos a conocer un poco más sobre redes. En español, significa Calidad de Servicio y muy probablemente, esto se explique por sí solo. Sin embargo, es bueno saber qué podemos obtener de esta característica de las redes. QoS consiste en una métrica o una serie de métricas que nos informan respecto a la calidad de la comunicación que se da en la red. Una de las informaciones que podemos conseguir gracias a QoS es si nuestro proveedor de servicios de Internet (ISP) está proveyendo el ancho de banda prometido.

 
Quality of Service (QoS) y Class of Service (CoS) van de la mano a la hora de garantizar alta calidad de transmisión de datos a la red. Muy probablemente pienses que simplemente estos conceptos deben integrarse y denominarse únicamente QoS. En resumen: QoS es la estrategia o la información que nos ayuda a identificar cuáles son las acciones a realizar para que la red funcione de la manera deseada. CoS son las acciones en cuestión propuestas por QoS, como el ejemplo que hemos citado: el establecimiento de prioridad de tráfico en la red basado en las aplicaciones que se consideran más críticas dentro de una organización.


Cuando Kubernetes crea un Pod, asigna una de estas clases de QoS al Pod:

* Guaranteed
* Burstable
* BestEffort
 

### Pod de QoS de Garantizado (Guaranteed)
Para que un Pod reciba una clase de QoS de Garantizado:

Cada contenedor en el Pod debe tener un límite de memoria y una solicitud de memoria, y deben ser los mismos.
Cada contenedor en el Pod debe tener un límite de CPU y una solicitud de CPU, y deben ser iguales.
Aquí está el archivo de configuración para un Pod que tiene un Contenedor. El contenedor tiene un límite de memoria y una solicitud de memoria, ambos equivalentes a 200 MiB. El contenedor tiene un límite de CPU y una solicitud de CPU, ambos equivalentes a 700 miliCPU:

```
      limits:
        memory: "200Mi"
        cpu: "700m"
      requests:
        memory: "200Mi"
        cpu: "700m"
```
Tenemos un ejemplo en pod-qos-guaranteed.yaml.
 
### Pod de QoS de Burstable  

Un Pod recibe una clase de QoS de Burstable si:

* El Pod no cumple los criterios para la clase de QoS Garantizada.
* Al menos un Contenedor en el Pod tiene una solicitud de memoria o CPU.

Aquí está el archivo de configuración para un Pod que tiene un Contenedor. El contenedor tiene un límite de memoria de 200 MiB y una solicitud de memoria de 100 MiB.
```
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"
```
Tenemos un ejemplo en pod-qos-burstable.yaml.

 
### Pod de QoS de BestEffort  
Para que un Pod reciba una clase de QoS de BestEffort, los Contenedores en el Pod no deben tener ningún límite o solicitud de memoria o CPU.

Tenemos un ejemplo en pod-qos-besteffort.yaml.


