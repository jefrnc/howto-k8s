[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/josephefranco)

# Introduccion a los Replication Controller

Resumen

- [ ] TBD
- [ ] Revision de esta guia (esto se escribio de forma rapida, mientras aprendia la plataforma con lo cual pueden existir correcciones o mejoras. No dudes de hacerme un PR para ir mejorando el contenido)


## ¿Qué es un Replication Controller?

El Controlador de replicación es la forma original de replicación en Kubernetes. Se está reemplazando por conjuntos de réplica (replicaSets), pero todavía se usa ampliamente, por lo que vale la pena entender qué es y cómo funciona. 
Un Controlador de replicación es una estructura que le permite crear fácilmente varios pods y luego asegurarse de que ese número de pods siempre exista. Si un pod se bloquea, el Controlador de replicación lo reemplaza. Los Controladores de replicación también proporcionan otros beneficios, como la capacidad de escalar el número de pods y de actualizar o eliminar múltiples pods con un solo comando. Puede crear un Controlador de replicación con un comando imperativo, o declarativamente, desde un archivo. Por ejemplo, cree un nuevo archivo llamado rc.yaml y agregue el siguiente texto:

```
$ kubectl apply -f rc.yaml
replicationcontroller/my-custom-app-rct created
```

```
$ kubectl describe rc my-custom-app-rct

Name:         my-custom-app-rct
Namespace:    default
Selector:     app=my-custom-app-rct
Labels:       app=my-custom-app-rct
Annotations:  Replicas:  3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=my-custom-app-rct
  Containers:
   my-custom-app-rct:
    Image:      python:3.6-alpine
    Port:       8080/TCP
    Host Port:  0/TCP
    Command:
      sh
      -c
      echo running > index.html && python -m http.server 8080
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                    Message
  ----    ------            ----  ----                    -------
  Normal  SuccessfulCreate  93s   replication-controller  Created pod: my-custom-app-rct-jlkb6
  Normal  SuccessfulCreate  93s   replication-controller  Created pod: my-custom-app-rct-vkbmh
  Normal  SuccessfulCreate  93s   replication-controller  Created pod: my-custom-app-rct-nj7mm
```

```
$ kubectl get pods | grep my-custom-app

my-custom-app-rct-jlkb6   1/1     Running   0          2m57s
my-custom-app-rct-nj7mm   1/1     Running   0          2m57s
my-custom-app-rct-vkbmh   1/1     Running   0          2m57s
```

```
$ kubectl delete rc my-custom-app-rct
replicationcontroller "my-custom-app-rct" deleted
```

```
$kubectl get pods | grep my-custom-app
my-custom-app-rct-jlkb6   1/1     Terminating   0          4m25s
my-custom-app-rct-nj7mm   1/1     Terminating   0          4m25s
my-custom-app-rct-vkbmh   1/1     Terminating   0          4m25s

$kubectl get pods | grep my-custom-app
```


*************

 


rs.yaml

 
Avancemos y creemos el conjunto de réplicas y echemos un vistazo:


# kubectl create -f replicaset.yaml
replicaset "soaktestrs" created

# kubectl describe rs soaktestrs
Name:           soaktestrs
Namespace:      default
Image(s):       nickchase/soaktest
Selector:       app in (soaktest,soaktestrs),teir notin (production)
Labels:         app=soaktestrs
Replicas:       3 current / 3 desired
Pods Status:    3 Running / 0 Waiting / 0 Succeeded / 0 Failed
No volumes.
Events:
  FirstSeen     LastSeen        Count   From                            SubobjectPath   Type    Reason                   Message
  ---------     --------        -----   ----                            -------------   --------------                   -------
  1m            1m              1       {replicaset-controller }                        Normal  SuccessfulCreate Created pod: soaktestrs-it2hf
  1m            1m              1       {replicaset-controller }                       Normal  SuccessfulCreate Created pod: soaktestrs-kimmm
  1m            1m              1       {replicaset-controller }                        Normal  SuccessfulCreate Created pod: soaktestrs-8i4ra

# kubectl get pods
NAME               READY     STATUS    RESTARTS   AGE
soaktestrs-8i4ra   1/1       Running   0          1m
soaktestrs-it2hf   1/1       Running   0          1m
soaktestrs-kimmm   1/1       Running   0          1m


As you can see, the output is pretty much the same as for a Replication Controller (except for the selector), and for most intents and purposes, they are similar.  The major difference is that the rolling-update command works with Replication Controllers, but won’t work with a Replica Set.  This is because Replica Sets are meant to be used as the backend for Deployments. Let’s clean up before we move on.


# kubectl delete rs soaktestrs
replicaset "soaktestrs" deleted

# kubectl get pods
Again, the pods that were created are deleted when we delete the Replica Set.

*************



Deployments
Deployments are intended to replace Replication Controllers.  They provide the same replication functions (through Replica Sets) and also the ability to rollout changes and roll them back if necessary. Let’s create a simple Deployment using the same image we’ve been using.  First create a new file, deployment.yaml, and add the following:


deployment.yaml


Now go ahead and create the Deployment:
# kubectl create -f deployment.yaml
deployment "soaktest" created
Now let’s go ahead and describe the Deployment:
# kubectl describe deployment soaktest
Name:                   soaktest
Namespace:              default
CreationTimestamp:      Sun, 05 Mar 2017 16:21:19 +0000
Labels:                 app=soaktest
Selector:               app=soaktest
Replicas:               5 updated | 5 total | 5 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
OldReplicaSets:         <none>
NewReplicaSet:          soaktest-3914185155 (5/5 replicas created)
Events:
  FirstSeen     LastSeen        Count   From                            SubobjectPath   Type    Reason                   Message
  ---------     --------        -----   ----                            -------------   --------------                   -------
  38s           38s             1       {deployment-controller }                        Normal  ScalingReplicaSet        Scaled up replica set soaktest-3914185155 to 3
  36s           36s             1       {deployment-controller }                        Normal  ScalingReplicaSet        Scaled up replica set soaktest-3914185155 to 5
As you can see, rather than listing the individual pods, Kubernetes shows us the Replica Set.  Notice that the name of the Replica Set is the Deployment name and a hash value. A complete discussion of updates is out of scope for this article — we’ll cover it in the future — but couple of interesting things here:
The StrategyType is RollingUpdate. This value can also be set to Recreate.
By default we have a minReadySeconds value of 0; we can change that value if we want pods to be up and running for a certain amount of time — say, to load resources — before they’re truly considered “ready”.
The RollingUpdateStrategy shows that we have a limit of 1 maxUnavailable — meaning that when we’re updating the Deployment, we can have up to 1 missing pod before it’s replaced, and 1 maxSurge, meaning we can have one extra pod as we scale the new pods back up.
As you can see, the Deployment is backed, in this case, by Replica Set soaktest-3914185155. If we go ahead and look at the list of actual pods…
# kubectl get pods
NAME                        READY     STATUS    RESTARTS   AGE
soaktest-3914185155-7gyja   1/1       Running   0          2m
soaktest-3914185155-lrm20   1/1       Running   0          2m
soaktest-3914185155-o28px   1/1       Running   0          2m
soaktest-3914185155-ojzn8   1/1       Running   0          2m
soaktest-3914185155-r2pt7   1/1       Running   0          2m
… you can see that their names consist of the Replica Set name and an additional identifier.



***

Scaling up or down: Manually changing the number of replicas
One common task is to scale up a Deployment in response to additional load. Kubernetes has autoscaling, but we’ll talk about that in another article.  For now, let’s look at how to do this task manually. The most straightforward way is to simply use the scale command:
# kubectl scale --replicas=7 deployment/soaktest
deployment "soaktest" scaled






***

Deploying a new version: Replacing replicas by changing their label
Another way you can use deployments is to make use of the selector.  In other words, if a Deployment controls all the pods with a tier value of dev, changing a pod’s teir label to prod will remove it from the Deployment’s sphere of influence. This mechanism enables you to selectively replace individual pods. For example, you might move pods from a dev environment to a production environment, or you might do a manual rolling update, updating the image, then removing some fraction of pods from the Deployment; when they’re replaced, it will be with the new image. If you’re happy with the changes, you can then replace the rest of the pods. Let’s see this in action.  As you recall, this is our Deployment:
# kubectl describe deployment soaktest
Name:                   soaktest
Namespace:              default
CreationTimestamp:      Sun, 05 Mar 2017 19:31:04 +0000
Labels:                 app=soaktest
Selector:               app=soaktest
Replicas:               3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
OldReplicaSets:         <none>
NewReplicaSet:          soaktest-3869910569 (3/3 replicas created)
Events:
  FirstSeen     LastSeen        Count   From                            SubobjectPath   Type              Reason                  Message
  ---------     --------        -----   ----                            -------------   --------  ------                  -------
  50s           50s             1       {deployment-controller }                        Normal            ScalingReplicaSet       Scaled up replica set soaktest-3869910569 to 3
And these are our pods:
# kubectl describe replicaset soaktest-3869910569
Name:           soaktest-3869910569
Namespace:      default
Image(s):       nickchase/soaktest
Selector:       app=soaktest,pod-template-hash=3869910569
Labels:         app=soaktest
                pod-template-hash=3869910569
Replicas:       5 current / 5 desired
Pods Status:    5 Running / 0 Waiting / 0 Succeeded / 0 Failed
No volumes.
Events:
  FirstSeen     LastSeen        Count   From                            SubobjectPath   Type              Reason                  Message
  ---------     --------        -----   ----                            -------------   --------  ------                  -------
  2m            2m              1       {replicaset-controller }                        Normal            SuccessfulCreate        Created pod: soaktest-3869910569-0577c
  2m            2m              1       {replicaset-controller }                        Normal            SuccessfulCreate        Created pod: soaktest-3869910569-wje85
  2m            2m              1       {replicaset-controller }                        Normal            SuccessfulCreate        Created pod: soaktest-3869910569-xuhwl
  1m            1m              1       {replicaset-controller }                        Normal            SuccessfulCreate        Created pod: soaktest-3869910569-8cbo2
  1m            1m              1       {replicaset-controller }                        Normal            SuccessfulCreate        Created pod: soaktest-3869910569-pwlm4
We can also get a list of pods by label:
# kubectl get pods -l app=soaktest
NAME                        READY     STATUS    RESTARTS   AGE
soaktest-3869910569-0577c   1/1       Running   0          7m
soaktest-3869910569-8cbo2   1/1       Running   0          6m
soaktest-3869910569-pwlm4   1/1       Running   0          6m
soaktest-3869910569-wje85   1/1       Running   0          7m
soaktest-3869910569-xuhwl   1/1       Running   0          7m
So those are our original soaktest pods; what if we wanted to add a new label?  We can do that on the command line:
# kubectl label pods soaktest-3869910569-xuhwl experimental=true
pod "soaktest-3869910569-xuhwl" labeled

# kubectl get pods -l experimental=true
NAME                        READY     STATUS    RESTARTS   AGE
soaktest-3869910569-xuhwl   1/1       Running   0          14m
So now we have one experimental pod.  But since the experimental label has nothing to do with the selector for the Deployment, it doesn’t affect anything. So what if we change the value of the app label, which the Deployment is looking at?
# kubectl label pods soaktest-3869910569-wje85 app=notsoaktest --overwrite
pod "soaktest-3869910569-wje85" labeled
In this case, we need to use the overwrite flag because the app label already exists. Now let’s look at the existing pods.
# kubectl get pods
NAME                        READY     STATUS    RESTARTS   AGE
soaktest-3869910569-0577c   1/1       Running   0          17m
soaktest-3869910569-4cedq   1/1       Running   0          4s
soaktest-3869910569-8cbo2   1/1       Running   0          16m
soaktest-3869910569-pwlm4   1/1       Running   0          16m
soaktest-3869910569-wje85   1/1       Running   0          17m
soaktest-3869910569-xuhwl   1/1       Running   0          17m
As you can see, we now have six pods instead of five, with a new pod having been created to replace *wje85, which was removed from the deployment. We can see the changes by requesting pods by label:
# kubectl get pods -l app=soaktest
NAME                        READY     STATUS    RESTARTS   AGE
soaktest-3869910569-0577c   1/1       Running   0          17m
soaktest-3869910569-4cedq   1/1       Running   0          20s
soaktest-3869910569-8cbo2   1/1       Running   0          16m
soaktest-3869910569-pwlm4   1/1       Running   0          16m
soaktest-3869910569-xuhwl   1/1       Running   0          17m
Now, there is one wrinkle that you have to take into account; because we’ve removed this pod from the Deployment, the Deployment no longer manages it.  So if we were to delete the Deployment…
# kubectl delete deployment soaktest
deployment "soaktest" deleted
The pod remains:
# kubectl get pods
NAME                        READY     STATUS    RESTARTS   AGE
soaktest-3869910569-wje85   1/1       Running   0          19m
You can also easily replace all of the pods in a Deployment using the –all flag, as in:
# kubectl label pods --all app=notsoaktesteither --overwrite
But remember that you’ll have to delete them all manually!


otra fuente
//https://www.josedomingo.org/pledin/2018/07/recursos-de-kubernetes-replicaset/