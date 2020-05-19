[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/josephefranco)

# Introduccion a los Deployments

Resumen

- [ ] TBD
- [ ] Revision de esta guia (esto se escribio de forma rapida, mientras aprendia la plataforma con lo cual pueden existir correcciones o mejoras. No dudes de hacerme un PR para ir mejorando el contenido)


## ¿Qué es un Namespace?

Los espacios de nombres están pensados para utilizarse en entornos con muchos usuarios distribuidos entre múltiples equipos, o proyectos. Para aquellos clústeres con unas pocas decenas de usuarios, no deberías necesitar crear o pensar en espacios de nombres en absoluto. Empieza a usarlos solamente si necesitas las características que proporcionan.

Los espacios de nombres proporcionan un campo de acción para los nombres. Los nombres de los recursos tienen que ser únicos dentro de cada espacio de nombres, pero no entre dichos espacios de nombres.

Los espacios de nombres son una forma de dividir los recursos del clúster entre múltiples usuarios (via cuotas de recursos).

En futuras versiones de Kubernetes, los objetos de un mismo espacio de nombres tendrán las mismas políticas de control de acceso por defecto.

No es necesario usar múltiples espacios de nombres sólo para separar recursos ligeramente diferentes, como versiones diferentes de la misma aplicación: para ello utiliza etiquetas para distinguir tus recursos dentro del mismo espacio de nombres


Kubernetes arranca con tres espacios de nombres inicialmente:

+ default El espacio de nombres por defecto para aquellos objetos que no especifican ningún espacio de nombres
+ kube-system El espacio de nombres para aquellos objetos creados por el sistema de Kubernetes
+ kube-public Este espacio de nombres se crea de forma automática y es legible por todos los usuarios (incluyendo aquellos no autenticados). Este espacio de nombres se reserva principalmente para uso interno del clúster, en caso de que algunos recursos necesiten ser visibles y legibles de forma pública para todo el clúster. La naturaleza pública de este espacio de nombres es simplemente por convención, no es un requisito

Mas informacion en https://kubernetes.io/es/docs/concepts/overview/working-with-objects/namespaces/

## DNS
Cuando creas un Servicio, se crea una entrada DNS correspondiente. Esta entrada tiene la forma <service-name>.<namespace-name>.svc.cluster.local, que significa que si un contenedor simplemente usa <service-name>, se resolverá al servicio que sea local al espacio de nombres. Esto es de utilidad para poder emplear la misma configuración entre múltiples espacios de nombres como Development, Staging y Production. Si quieres referenciar recursos entre distintos espacios de nombres, entonces debes utilizar el nombre cualificado completo de dominio (FQDN).

 
## Creacion de un Namespace

Primero verificamos cuales tenemos funcionando
```
$ kubectl get ns

NAME              STATUS   AGE
default           Active   21h
kube-node-lease   Active   21h
kube-public       Active   21h
kube-system       Active   21h
```

Aplicamos el manifiesto ns.yaml y verificamos que se haya creado el mismo
```
$ kubectl apply -f ns.yaml
namespace/development created

$ kubectl get ns

NAME              STATUS   AGE
default           Active   21h
development       Active   3s
kube-node-lease   Active   21h
kube-public       Active   21h
kube-system       Active   21h
```

Cuando iteractuamos en el kubectl todos las operaciones se hacen en el cnotexto actual que nos encontramos y al namespace asociado, si quisieramos cambiar al nuevo namespace deberiamos hacer lo siguiente

```
$ kubectl config set-context --current --namespace=development
```

Sino en su defecto vamos a tener que especificarlo en la linea de comando

```
$ kubectl get pods --namespace development
```
  

## Creacion de un Namespace para entornos

si vemos el yaml envs-ns.yaml, creamos dos ns, uno de dev y otro de prod. La idea es poder tener aisladas las aplicaciones por ns.


```
$ kubectl apply -f envs-ns.yaml

namespace/dev created
namespace/prod created
deployment.apps/deployment-dev created
deployment.apps/deployment-prod created
```

Verificamos el entorno de Dev

```
$ kubectl get pods --namespace dev

NAME                              READY   STATUS    RESTARTS   AGE
deployment-dev-64588d8b49-djpkr   1/1     Running   0          22s
```

Verificamos el entorno de Prod
```
$ kubectl get pods --namespace prod

NAME                               READY   STATUS    RESTARTS   AGE
deployment-prod-6d4c767fcc-5vmss   1/1     Running   0          27s
deployment-prod-6d4c767fcc-cgkm4   1/1     Running   0          27s
deployment-prod-6d4c767fcc-hhmkj   1/1     Running   0          27s
deployment-prod-6d4c767fcc-nkrh4   1/1     Running   0          27s
deployment-prod-6d4c767fcc-wfvcc   1/1     Running   0          27s
```


## Creacion de NS y como obtener el DNS


Para eso nos basamos en otro ejemplo
```
$ kubectl apply -f ns-and-dns.yaml 

namespace/ci created
deployment.apps/backend-k8s-test created
service/backend-k8s-test created
```

Verificamos que se haya creando el nuevo ns que se llama ci.

```
$ kubectl get ns

NAME              STATUS   AGE
ci                Active   11s
default           Active   21h
dev               Active   11m
development       Active   19m
kube-node-lease   Active   21h
kube-public       Active   21h
kube-system       Active   21h
prod              Active   11m
```

Verificamos que existan los nuevos objetos en este ns.

```
$ kubectl get all -n ci 

NAME                                    READY   STATUS    RESTARTS   AGE
pod/backend-k8s-test-867ffc6f87-g55lc   1/1     Running   0          82s
pod/backend-k8s-test-867ffc6f87-lnlzz   1/1     Running   0          82s
pod/backend-k8s-test-867ffc6f87-pfng7   1/1     Running   0          82s

NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/backend-k8s-test   ClusterIP   10.110.210.138   <none>        80/TCP    82s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/backend-k8s-test   3/3     3            3           82s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/backend-k8s-test-867ffc6f87   3         3         3       82s
```

Describimos el servicio creado


```
$ kubectl describe svc backend-k8s-test -n ci

Name:              backend-k8s-test
Namespace:         ci
Labels:            app=backend
Annotations:       Selector:  app=backend
Type:              ClusterIP
IP:                10.110.210.138
Port:              <unset>  80/TCP
TargetPort:        8080/TCP
Endpoints:         172.17.0.30:8080,172.17.0.31:8080,172.17.0.32:8080
Session Affinity:  None
Events:            <none>

```

La estructura de DNS es la siguiente  {Name}.{Namespace}.svc.cluster.local entonces quedaria como backend-k8s-test.ci.svc.cluster.local, volvemos a crear un pod para probarlo.

```
$ kubectl run  -it --rm terminal-temp  --image=praqma/network-multitool  -- sh      

If you don't see a command prompt, try pressing enter.
/ # curl http://backend-k8s-test.ci.svc.cluster.local
test

```

Y vemos que funciono.