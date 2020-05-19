[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/josephefranco)

# Introduccion a los Deployments

Resumen

- [ ] TBD
- [ ] Revision de esta guia (esto se escribio de forma rapida, mientras aprendia la plataforma con lo cual pueden existir correcciones o mejoras. No dudes de hacerme un PR para ir mejorando el contenido)


## ¿Qué es un Deployment?

El despliegue de un Deployment crea un ReplicaSet y los Pods correspondientes. Por lo tanto en la definición de un Deployment se define también el replicaSet asociado. En la práctica siempre vamos a trabajar con Deployment. Los atributos relacionados con el Deployment que hemos indicado en la definición son:

* revisionHistoryLimit: Indicamos cuántos ReplicaSets antiguos deseamos conservar, para poder realizar rollback a estados anteriores. Por defecto, es 10.
* strategy: Indica el modo en que se realiza una actualización del Deployment: recreate: elimina los Pods antiguos y crea los nuevos; RollingUpdate: actualiza los Pods a la nueva versión.



```
$ kubectl apply -f dep.yaml
deployment.apps/deployment-test created
```

```
$ kubectl  get deploy  deployment-test -o yaml | grep ReplicaSet
    message: ReplicaSet "deployment-test-64588d8b49" has successfully progressed.
```

```
$ kubectl get rs deployment-test-64588d8b49

NAME                         DESIRED   CURRENT   READY   AGE
deployment-test-64588d8b49   3         3         3       2m5s
```

 

```
$ kubectl get pods 

NAME                               READY   STATUS    RESTARTS   AGE
deployment-test-64588d8b49-2kf25   1/1     Running   0          6m40s
deployment-test-64588d8b49-8jtwf   1/1     Running   0          6m40s
deployment-test-64588d8b49-qm62d   1/1     Running   0          6m40s

```

 


```
$ kubectl get rs

NAME                         DESIRED   CURRENT   READY   AGE
deployment-test-64588d8b49   0         0         0       9m56s
deployment-test-6b86b59448   3         3         3       17s
```


```
$ kubectl get pods

NAME                               READY   STATUS    RESTARTS   AGE
deployment-test-6b86b59448-m9dc8   1/1     Running   0          52s
deployment-test-6b86b59448-thfww   1/1     Running   0          54s
deployment-test-6b86b59448-xqlh2   1/1     Running   0          51s
```



```
$ kubectl rollout undo deployment.apps/deployment-test
deployment.apps/deployment-test rolled back
```


PS C:\Repos\k8s-bootcamp\deployment> kubectl get pods

```
NAME                               READY   STATUS    RESTARTS   AGE
deployment-test-64588d8b49-mnp9v   1/1     Running   0          38s
deployment-test-64588d8b49-nlr55   1/1     Running   0          37s
deployment-test-64588d8b49-tgxrt   1/1     Running   0          39s
```


kubectl run  -it --rm terminal-temp  --image=praqma/network-multitool  -- sh 