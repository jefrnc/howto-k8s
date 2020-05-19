[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/josephefranco)

# Introduccion a los Deployments

Resumen

- [ ] TBD
- [ ] Revision de esta guia (esto se escribio de forma rapida, mientras aprendia la plataforma con lo cual pueden existir correcciones o mejoras. No dudes de hacerme un PR para ir mejorando el contenido)
 
## Introduccion
Ahora vemos un ejemplo simple a definir resource quota a un nuevo namespace denominado uat.

En este ejemplo vamos a usar el archivo res-quota.yaml

```
$ kubectl apply -f res-quota.yaml

namespace/uat created
resourcequota/res-quota created
deployment.apps/deployment-test created
```


```
$ kubectl describe resourcequotas res-quota -n uat 

Name:            res-quota
Namespace:       uat
Resource         Used  Hard
--------         ----  ----
limits.cpu       1     2
limits.memory    1G    2Gi
requests.cpu     1     1
requests.memory  1G    1Gi
```

Ya que llegamos al limite maximo, si aumentamos la cantidad de replicas a tres y volvemos a aplicar el manifiesto vamos a ver al consulta el deployment

```
$ kubectl get deployments.app -n uat deployment-test 

NAME              READY   UP-TO-DATE   AVAILABLE   AGE
deployment-test   2/3     2            2           54s
```
Hay uno de los pods que no los pudo levantar


```
$ kubectl get deployments.app -n uat deployment-test -o yaml | grep message
    message: Deployment does not have minimum availability.
    message: 'pods "deployment-test-799f96d6cf-4whfp" is forbidden: exceeded quota:
    message: ReplicaSet "deployment-test-799f96d6cf" is progressing.
```

Entonces nos indica que no se pudo crear por falta de cuota.


# Restriccion a nivel de cantidad de objetos

Nos permite setear cuotas a nivel de cantidad de pods, por ejemplo que solo dejemos que existen 3 pods en un namespace.
Para eso podemos usar el ejemplo pod-quota.yaml

```
$ kubectl apply -f pod-quota.yaml

namespace/qa unchanged
resourcequota/pod-demo unchanged
deployment.apps/deployment-qa configured
```

Verificamos la cuota que generamos

```
$ kubectl get resourcequota pod-demo -n qa

NAME       AGE   REQUEST     LIMIT
pod-demo   93s   pods: 3/3
```

Verificamos los pods que estan corriendo
```
$ kubectl get pods -n qa

NAME                             READY   STATUS    RESTARTS   AGE
deployment-qa-64588d8b49-2rbwm   1/1     Running   0          111s
deployment-qa-64588d8b49-cfsxx   1/1     Running   0          111s
deployment-qa-64588d8b49-xbhf7   1/1     Running   0          111s
```

Como veran no esta funcionando la replica segun lo que configuramos, asi que vamos a revisar el deployment.
```
$ kubectl get deployments.app -n qa deployment-qa -o yaml | grep message

            f:message: {}
            f:message: {}
            f:message: {}
    message: 'pods "deployment-qa-64588d8b49-gfqnw" is forbidden: exceeded quota:
    message: ReplicaSet "deployment-qa-64588d8b49" is progressing.
    message: Deployment does not have minimum availability.
```
Encontramos un error similar al caso anterior cuando superamos la cuota, no se pudo crear todos los objetos solicitados. 

```
$ kubectl get rs -n qa                                        

NAME                       DESIRED   CURRENT   READY   AGE
deployment-qa-64588d8b49   10        3         3       3m8s
```

Cuando revisamos el replica set nos indica que eran 10 pods previsto y solo tenemos operativos 3.