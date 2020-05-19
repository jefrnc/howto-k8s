[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/josephefranco)

# Bootcamp de Kubernetes (K8s)

Este repositorio fue armado como un entrenamiento rapido en Kubernetes.

## Comenzando 🚀

Abajo de este README, vas a encontrar lo pasos para crear el ambiente en minikube en caso de no contar con un cluster de Kubernetes.

## Roadmap

### ¿Que son los  Pods?
Introduccion a la unidad minima de deployment dentro de Kubernetes [Click aqui](pods/README.md).

---
### ¿Para qué sirve la replicación de Kubernetes?

Antes de analizar cómo haría la replicación, hablemos sobre por qué. Por lo general, desearía replicar sus contenedores (y, por lo tanto, sus aplicaciones) por varias razones, que incluyen:
* Fiabilidad: al tener varias versiones de una aplicación, evita problemas si falla uno o más. 
* Equilibrio de carga: tener múltiples versiones de un contenedor le permite enviar fácilmente tráfico a diferentes instancias para evitar la sobrecarga de una sola instancia o nodo. Esto es algo que Kubernetes hace out-of-the-box, lo que lo hace extremadamente conveniente.
* Escalado: cuando la carga se vuelve demasiado para la cantidad de instancias existentes, Kubernetes le permite escalar fácilmente su aplicación, agregando instancias adicionales según sea necesario.

La replicación es apropiada para numerosos casos de uso, que incluyen:

* Aplicaciones basadas en microservicios: en estos casos, múltiples aplicaciones pequeñas proporcionan una funcionalidad muy específica.
* Aplicaciones nativas de la nube: debido a que las aplicaciones nativas de la nube se basan en la teoría de que cualquier componente puede fallar en cualquier momento, la replicación es un entorno perfecto para implementarlas, ya que múltiples instancias están integradas en la arquitectura.
* Aplicaciones móviles: las aplicaciones móviles a menudo se pueden diseñar para que el cliente móvil interactúe con una versión aislada de la aplicación del servidor.

Kubernetes tiene varias formas de implementar la replicación.

* Replication Controller
* Replica Sets
* Deployments

En los proximos pasos vamos a profundizar en cada uno.

---
### Construyendo un Replication Controller
Entra a esta practica haciendo [click aqui](replicationController/README.md)

---
### Construyendo un Replica Sets
Entra a esta practica haciendo [click aqui](replicaSet/README.md)

---
### Construyendo un Deployments
Entra a esta practica haciendo [click aqui](deployment/README.md)

---
### Construyendo un Job
Entra a esta practica haciendo [click aqui](job/README.md)

---
### El uso de Service & Endpoints
Entra a esta practica haciendo [click aqui](service/README.md)

---
### El uso de namespaces
Entra a esta practica haciendo [click aqui](namespaces/README.md)

---
### Limitar Ram y CPU

La idea es limitar un pod que no pueda usar mas de X cantidad de Ram o CPU, la idea es que el nodo no supere la cantidad de recursos disponibles ya que ellos podrian hacer caer el nodo.

Entra a esta practica haciendo [click aqui](limits-request/README.md)

---
### El uso de LimitRange
 
Aplicar politica a pods que no tengan limit/request a un Namespace .
Tambien podemos definir max o minimos a nivel de Namespace.

Entra a esta practica haciendo [click aqui](limitRange/README.md)
---
### El uso de Resource Quota
 
Limite Range | Resource Quota
------------ | -------------
Las politicas aplica a nivel de Pod, por lo cual si pongo que los limites de CPU son 1Cpu, al tener por ejemplo 5 pods corriendo podria estar en una carga de 5cpu para ese nodo. Es ideal para politicas de Pods, pero por ejemplo si tenemos replicas vemos impactada la carga en el nodo ya que las policitas aplico a la unidad del Pod. | No actua a nivel de objeto, sino a nivel de Namespace. Especifica por ejemplo maximo 3cpu por Namespace, quiere decir que la sumatoria de toda los cpu o memoria de los pods que se encuentra en ese namespace no puede seperar los limites establecido por el resource quota.

Por lo que se ve viene a complementar las politicas que se definieron a limite range.


Entra a esta practica haciendo [click aqui](resourceQuota/README.md)

---
### El uso de Probes
 
Como k8s sabe que un pod esta listo para funcionar? y que continua funcionando bien? Bueno entra a esta practica haciendo [click aqui](resourceQuota/README.md).

---
### configMaps
 
Bueno entra a esta practica haciendo [click aqui](configmaps/README.md).

---
### secrets
 
Bueno entra a esta practica haciendo [click aqui](secrets/README.md).

---
### Network Policies
 
Bueno entra a esta practica haciendo [click aqui](networkpolicy/README.md).

---
### ingress
TBD
---
### volumes
TBD
---
### RBAC
TBD
---
### Rancher
TBD
---
 

## Entorno de minikube

Minikube es una herramienta que administra maquinas virtuales en donde corre un cluster o mejor dicho una instancia de Kubernetes en un solo nodo, usualmente lo usan con VirtualBox pero en mi caso fue con HyperV.

https://kubernetes.io/es/docs/tasks/tools/install-minikube/

Para levantarlo
```
$ minikube start
```

Si queremos destruir nuestro deployment, o en su defecto toda la instancia.
```
$ kubectl delete deployment --all
$ minikube delete
```

## Gestion de contextos

Podemos revisar la configuracion de nuestro cliente de kubectl

```
$ kubectl config view
```
 
Revisar en que contexto nos encontramos trabajando
```
$ kubectl config current-context 
```

Ver los distintos contextos que tenemos en el equipo
```
$ kubectl config get-contexts
CURRENT   NAME       CLUSTER    AUTHINFO                                      NAMESPACE
*         AKS-Test   AKS-Test   clusterUser_DevOpsLabs-AKS-Test-RG_AKS-Test
          aks        aks        clusterUser_DevOpsLab-Prod-RG_aks
          aks-test   aks-test   clusterUser_DevOpsLab-Test-RG_aks-test
          minikube   minikube   minikube
```
          
 
Cambiar de contexto
```
$ kubectl config use-context minikube
Switched to context "minikube".
```


## Creacion de objetos

```
$ kubectl apply -f ./my-manifest.yaml            # create resource(s)
$ kubectl apply -f ./my1.yaml -f ./my2.yaml      # create from multiple files
$ kubectl apply -f ./dir                         # create resource(s) in all manifest files in dir
$ kubectl apply -f https://git.io/vPieo          # create resource(s) from url
$ kubectl create deployment nginx --image=nginx  # start a single instance of nginx
$ kubectl explain pods                           # get the documentation for pod manifests
```

## Redireccion de Puertos
```
$ kubectk port-forward my-pod 8080:80
```

## Cursos recomendados

**Kubernetes para Desarrolladores**
https://www.udemy.com/course/kubernetes-para-desarrolladores/

 
## Proximos temas a investigar

- [ ] pods presets
- [ ] affinity
- [ ] webhooks
- [ ] CRDs 
- [ ] hacer una prueba del initcontainer
- [ ] pruebas para deployment
- [ ] pruebas para el rollback //rollout
- [ ] skaffold 
- [ ] cloud native development
- [ ] draft / azure
- [ ] tilt, garden y devspaces, otras alternativas
- [ ] debuggeear pods con el visual studio code
- [ ] network policies // a nivel de namespaces
- [ ] pods secutiry policites //limitar el tipo de contenedor
- [ ] srvice accounts
- [ ] CNCF - Cloud Native Interactive Landscape
- [ ] Service Mesh (istio)
- [ ] Open Tracing(nuready, jagger)
- [ ] CI/CD (Gitops - Argo / Flux Wewve / SPinanaker)
- [ ] Serverless (Openfass, Cloud Run - Knative)
 


## Links interesantes
- Conceptos de k8s : https://www.youtube.com/playlist?list=PLo5G9g9vTlqk21Bj8GObMcTBrDPdBjbQ2
- Curso KUBERNETES al completo : https://www.youtube.com/playlist?list=PLkqaOL-oB94HdBAzDSC-o6iUUsGog8DtK
- Docker / Kubernetes / Cloud : https://www.youtube.com/playlist?list=PLwH0tlWs8nkTQ8lNQ1usKML8pxAP4hEMH
- Okteto: https://okteto.com/
- Okteto Cloud: https://cloud.okteto.com
- Repo en Github: https://github.com/okteto/k8s-for-devs
- Implementing Advanced Scheduling Techniques with Kubernetes : https://blog.kublr.com/implementing-advanced-scheduling-techniques-with-kubernetes-a1d59aece87b
- Developing on Kubernetes : https://kubernetes.io/blog/2018/05/01/developing-on-kubernetes/
- 50+ Useful Kubernetes Tools : https://caylent.com/50-useful-kubernetes-tools
- https://medium.com/google-cloud/tagged/kubernetes
- https://engineering.opsgenie.com/advanced-kubernetes-objects-53f5e9bc0c28
- https://blenderfox.com/2018/01/23/how-to-move-from-single-master-to-multi-master-in-an-aws-kops-kubernetes-cluster/
- https://github.com/jose-franco/chaos-mesh
- https://docs.microsoft.com/en-us/azure/aks/kube-advisor-tool
- https://www.stackrox.com/post/2020/02/azure-kubernetes-aks-security-best-practices-part-2-of-4/
- https://dev.to/megatherion/using-azure-cli-and-powershell-to-create-kubernetes-cluster-in-azure-66n
- https://stackoverflow.com/questions/55035296/how-to-get-creator-information-about-particular-resource-from-kubectl-get-or-des

## Contribuyendo 🖇️

Por favor lee el [CONTRIBUTING.md]([CONTRIBUTING.md) para detalles de nuestro código de conducta, y el proceso para enviarnos pull requests.

 

## Licencia 📄

Mira el archivo [LICENSE.md](LICENSE.md) para detalles

## Expresiones de Gratitud 🎁

* Comenta a otros sobre este proyecto 📢
* Invita una cerveza 🍺 o un café ☕ a alguien del equipo. 
* Da las gracias públicamente 🤓.

---
⌨️ con ❤️ por [jose-franco](https://github.com/jose-franco) 😊