[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/josephefranco)

# Network policies

## ¿Qué es una Network Policies?

En criollo, una network policies termina siendo un firewall o filtrado de trafico en el cluster, que mediante los selectors se puede aplicar a ciertos objetos si tambien se lo desea.
Si nos fijamos que nos dice Google sobre esto, nos dice que la definición de políticas de red te ayuda a habilitar estrategias como la defensa en profundidad cuando el clúster entrega una aplicación de varios niveles. Por ejemplo, puedes crear una política de red para asegurar que un servicio de frontend vulnerable en la aplicación no pueda comunicarse directamente con un servicio de facturación o contabilidad en varios niveles inferiores.

La política de red también le facilita a la aplicación alojar datos de varios usuarios de manera simultánea. Por ejemplo, puedes proporcionar instancia múltiple segura si defines un modelo de instancia por espacio de nombres. En este modelo, las reglas de la política de red pueden garantizar que los pods y los servicios en un espacio de nombres determinado no puedan acceder a otros pods o servicios en un espacio de nombres diferente.


# Ejemplo simple

Esta NetworkPolicy eliminará todo el tráfico a los pods de una aplicación, seleccionada mediante Pod Selectores.

** En que caso usaria esto?: **
- Desea ejecutar un Pod y evitar que otros Pods se comuniquen con él.
- Desea aislar temporalmente el tráfico a un servicio de otros Pods.


Para esta primera prueba conceptural vamos a dejar corriendo un pod que vamos a intentar acceder desde otro pod.

Para eso levantamos al que vamos a acceder, con el nombre de web que simplemente es un servidor nginx.

```
$ kubectl run  web --image=nginx --labels app=web --expose --port 80
```


Ahora levantamos un pod solo para hacer un wget o curl a nuestro pod levantado previamente.

```
$ kubectl run --rm -i -t --image=alpine test-$RANDOM -- sh
/ # wget -qO- http://web
<!DOCTYPE html>
<html>
<head>
```

Como vemos, el acceso funciona, ahora aplicamos la politica de `netpol-web-deny-all.yaml`.

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: web-deny-all
spec:
  podSelector:
    matchLabels:
      app: web
  ingress: []
```
Una vez aplicada 

```sh
$ kubectl apply -f netpol-web-deny-all.yaml
networkpolicy "netpol-web-deny-all" created
```

Ejecute un contenedor de prueba nuevamente e intentamos consultar la web:

```
$ kubectl run --rm -i -t --image=alpine test-$RANDOM -- sh
/ # wget -qO- --timeout=2 http://web
wget: download timed out
```

Ya el trafico se encuentra filtrado y el pod no se encuentra accesible.

-----

### Observaciones

En el manifiesto anterior, apuntamos a Pods con la etiqueta `app=web` para vigilar la red. A este archivo de manifiesto le falta el campo `spec.ingress`. Por lo tanto, no está permitiendo ningún tráfico en el Pod.

Si crea otra NetworkPolicy que da acceso a algunos Pods a esta aplicación directa o indirectamente, esta NetworkPolicy quedará obsoleta.

Si hay al menos una NetworkPolicy con una regla que permite el tráfico, significa que el tráfico se enrutará al pod independientemente de las políticas que bloqueen el tráfico.

 
Hay muy buenas recetas de network policies en los siguientes links, les recomiendo que lo revisen.

- Ahmet Alp Balkan ([@ahmetb](https://twitter.com/ahmetb)).
- https://github.com/ahmetb/kubernetes-network-policy-recipes

Muchas gracias.