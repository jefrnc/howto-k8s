[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/josephefranco)

# Introduccion a la operatoria con Pods

Resumen

- [x] Correr nuestro primer pod
- [x] Creacion del primer pod por yaml
- [x] Varios contenedores en un mismo Pod 
- [x] Algunas cosas más
- [ ] Revision de esta guia (esto se escribio de forma rapida, mientras aprendia la plataforma con lo cual pueden existir correcciones o mejoras. No dudes de hacerme un PR para ir mejorando el contenido)


## ¿Qué es un pod?
Los pods son los objetos más pequeños y básicos que se pueden implementar en Kubernetes. Un pod representa una instancia única de un proceso en ejecución en el clúster.

Los pods comprenden uno o más contenedores, como los de Docker. Cuando un pod ejecuta varios contenedores, estos se administran como una sola entidad y comparten los recursos del pod. En general, ejecutar varios contenedores en un solo pod representa un caso práctico avanzado.

Los pods son efímeros, cuando se destruyen se pierde toda la información que contenía. Si queremos desarrollar aplicaciones persistentes tenemos que utilizar volúmenes (más adelante los veremos).

![Alt text](../asserts/pod01.png?raw=true "Title")

Te invito a leer este articulo que explaya muy bien de que se trata [clic aqui](https://cloud.google.com/kubernetes-engine/docs/concepts/pod?hl=es).

## Creacion de Pods

### Correr nuestro primer pod

```
$ kubectl run nginx --image=nginx --port=80 
```

### Creacion del primer pod por yaml

Creamos nuestro pod, el mismo lo tenemos en el archivo pod.yaml, mediante el comando kubectl apply aplicamos el manifiesto.

```
$kubectl apply -f pod.yaml
pod/my-pod created
```

Verificamos que el mismo se haya levantado, y esta ok.

```
$ kubectl get pods
NAME     READY   STATUS    RESTARTS   AGE
my-pod   1/1     Running   0          20s
```

Si queremos conectar al container de forma similar a la que haciamos con docker lo realizamos de la siguiente manera.
```
$kubectl exec -it my-pod -- sh
```

Si por algun motivo quisieramos ver el yaml de un pod deberiamos ejecutar lo siguiente
```
$ kubectl get pod my-pod -o yaml
```


Para borrarlo hacemos lo siguiente
```
$  kubectl delete -f pod.yaml
pod "my-pod" deleted
```

Si volvemos a revisar no tenemos ningun pod vigente.

```
$ kubectl get pods
No resources found in default namespace.
```


### Veamos otros ejemplos 

+ En un mismo yaml se puede crear varios pods, simplemente separados por **---**, aca queda otro ejemplo para aplicar.  [pod-multiples.yaml](pod-multiples.yaml)

+ Ya vimos como crear pods, pero como vieron son faciles de crear pero piensen que si tenemos muchos es complicado administrarlos o agruparlos eficazmente, por eso se puede utilizar los labels para insertar datos en la metadata. Veamos el ejemplo [pod-withlabels.yaml](pod-withlabels.yaml).
```
# Con estas etiquetas podemos buscar de forma facil y agil nuestros pods
$ kubectl get pods -l stage=dev,tier=front-end
NAME      READY   STATUS    RESTARTS   AGE
my-pod4   1/1     Running   0          8m20s
```

+ Otro ejemplo de labels en Pods. Veamos el ejemplo [pod-labels.yaml](pod-labels.yaml).  

+ Kubernetes también proporciona una lista de etiquetas recomendadas. Estas etiquetas se recomiendan para cada objeto que cree y tenga un prefijo compartido; app.kubernetes.io. Los prefijos aseguran que las etiquetas recomendadas no se mezclen con las etiquetas personalizadas definidas por los usuarios. Veamos el ejemplo [pod-withlabelsk8s.yaml](pod-withlabelsk8s.yaml).


+ Supongamos que tenemos un Pod que por alguna razon queremos que corra en un nodo linux (suponiendo que tenemos AKS) se puede usar un selector para alojarlo ahi. Veamos el [nginx.linuxnode.yaml](nginx.linuxnode.yaml).
 

### Varios contenedores en un mismo Pod 

Que pasa si tenemos dos contenedores en un pod? Veamos el ejemplo [pod-with2.yaml](pod-with2.yaml).
Tenemos dos instancias de nginx con distntos puertos pero en el mismo pod, tecnicamente no podrian escuchar en el mismo puerto por que comparte la misma interfaz de red y tendriamos conflictos al querer levantarlo.

Si verificamos el mismo deberia estar funcionando

```
# kubectl get pods | grep my-pod-twocontainer
```

Ahora que pasa si quisieramos ver los servidores de nginx que levantamos? Al estar en un cluster aunque sea virtualizado el segmento de IP asignado no es visible, nos queda solo dos caminos.. uno seria exponerlo para poder acceder (lo vamos a ver mas adelante) o crear un pod y validarlo desde adentro.

```
$ kubectl describe pods my-pod-twocontainer | grep IP
IP:           172.17.0.12
```

Para probar esto de forma simple, voy a basarme en una imagen praqma/network-multitool que contiene el curl, wget, ping, netcat, nslookup,host, dig, psql, mysql, swaks,etc.

Construimos un pod temporal para intentar consultar al pod que creamos con los dos contenedores:

```
$ kubectl run  -it --rm terminal-temp  --image=praqma/network-multitool  -- sh                                                                                               -- sh
If you don't see a command prompt, try pressing enter.
ls
bin                   home                  proc                  sys
certs                 lib                   root                  tmp
dev                   media                 run                   usr
docker-entrypoint.sh  mnt                   sbin                  var
etc                   opt                   srv
```

Pingueamos al Pod que creamos
```
$ ping 172.17.0.1
PING 172.17.0.1 (172.17.0.1) 56(84) bytes of data.
64 bytes from 172.17.0.1: icmp_seq=1 ttl=64 time=0.079 ms
64 bytes from 172.17.0.1: icmp_seq=2 ttl=64 time=0.080 ms
 
--- 172.17.0.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1044ms
rtt min/avg/max/mdev = 0.079/0.079/0.080/0.000 ms

```
Hacemos un curl al puerto de my-container1 

```
$ curl http://172.17.0.12:8082
my-container1

```

Hacemos un curl al puerto de my-container2
```
$ curl http://172.17.0.12:8083
my-container2

```
Salimos del contenedor
```
$ exit
Session ended, resume using 'kubectl attach terminal-temp -c terminal-temp -i -t' command when the pod is running
pod "terminal-temp" deleted
```
Vermeos que nuestro pod se destruyo automaticamente por el cmando -rm.




### Insertando variables de entorno en nuestro pod


Ejecutamos el ejemplo [pod-with-env-vars.yaml](pod-with-env-vars.yaml) el cual esta insertando dos variables mediante nuestro manifiesto y hacemos que la muestre un servidor web.

```
kubectl apply -f pod-with-env-vars.yaml 
pod/my-pod-with-env-vars created
```

Verificamos que el pod este levantado
```
kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
my-pod-with-env-vars   1/1     Running   0          8s
```

Buscamos la ip de nuestro pod
```
C:\Repos\k8s-bootcamp\_envs> kubectl get pods my-pod-with-env-vars -o yaml | grep ip
          k:{"ip":"172.17.0.15"}:
            f:ip: {}
  - ip: 172.17.0.15
```

Levantamos un pod dinamico para hacer un curl a nuestro server y vemos el resultado.
```
$ kubectl run  -it --rm terminal-temp  --image=praqma/network-multitool  -- sh 
If you don't see a command prompt, try pressing enter.
/ # curl  http://172.17.0.15:8080
el valor de var1 = valor de prueba 1 y var2 = estamos probando esto
```
Como veran esto es bastante util para insertar valores en nuestros pods.


### Insertando variables de referencia de nuestro Spect

Ejecutamos el ejemplo [pod-with-ref-vars.yaml](pod-with-ref-vars.yaml) 
```
$ kubectl apply -f pod-with-ref-vars.yaml
```

Verificamos que el pod este corriendo
```
$ kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
my-pod-with-ref-vars   1/1     Running   0          14s
```

Ontenemos la ip

```
$ kubectl get pods my-pod-with-ref-vars -o yaml | grep ip
  - ip: 172.17.0.16
```
Hacemos el curl para verificar el contenido

```
$ kubectl run  -it --rm terminal-temp  --image=praqma/network-multitool  -- sh 
If you don't see a command prompt, try pressing enter.
/ # curl http://172.17.0.16:8080
Estoy corriendo en el nodo minikube, mi nombre es my-pod-with-ref-vars y me dieron la ip 172.17.0.16/ # 
```

Como veran nos retorno los valores referenciados dinamicamentes en las variables de enterno. Muy bueno o no?

### Algunas cosas más

+ Si quisieramos borrar algun pod, lo hacemos de forma simple
```
kubectl delete pods my-pod
```
+ Que pasa si nuestro Pod se cae? Por ahora solo vimos como crear, actualizar y borrar. Para eso te invito a seguir viendo el proximo capitulo de [replicaSets](../replicaSet/README.md).
