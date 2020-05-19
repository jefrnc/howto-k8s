[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/josephefranco)

# Introducción al Limit Range

Resumen

## Introduccion

Son las mismas restricciones que nos permite los limit/request pero a nivel de namespace. Podes revisar el apartado de Limit/Request desde [aquí](../limits-request/README.md).
Por ejemplo seteamos en el ns de prod que el max de memoria es de 1Gi y CPU 1; y min 100M/100m.


```
  - max:
      memory: 1Gi
      cpu: 1
    min:
      memory: 100M
      cpu: 100m
```

En el mismo manifiesto vamos a intentar crear un Pod con 50m/50m.

```
$ kubectl apply -f min-max-limits.yaml

namespace/prod unchanged
limitrange/min-max configured
Error from server (Forbidden): error when creating "min-max-limits.yaml": pods "pod-limit-50m" is forbidden: [minimum cpu usage per Container is 100m, but request is 50m, minimum memory usage per Container is 100M, but request is 50M]
```
Como veran no lo permite crear por que se encuentra fuera de las especificaciones.

```
$ kubectl describe limits min-max --namespace=prod

Name:       min-max
Namespace:  prod
Type        Resource  Min   Max  Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---   ---  ---------------  -------------  -----------------------
Container   cpu       100m  1    1                1              -
Container   memory    100M  1Gi  1Gi              1Gi            -
```

Tambien se puede especificar valores por defecto a los pods que no contengan especificaciones, para eso esta el ejemplo de default-cpu-mem.yaml.

 
```
$ kubectl apply -f pod-test.yaml -n prod

pod/containers-test-limitrange created

$ kubectl get pods containers-test-limitrange --namespace=prod   
NAME                         READY   STATUS    RESTARTS   AGE
containers-test-limitrange   1/1     Running   0          3m43s

```


Podemos verificar que en la especificacion figuran estos limites.

```
$ kubectl get pods containers-test-limitrange --namespace=prod -o yaml | grep -C 6 resources
      resources:
      limits:
        cpu: 100m
        memory: 100M
      requests:
        cpu: 100m
        memory: 100M
```
