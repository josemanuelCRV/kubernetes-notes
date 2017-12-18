# Kubernetes



## Módulo 1 Crear un clúster de Kubernetes

***Visión general del módulo***

- aprende qué es un cluster de Kubernetes
- aprende qué es minikube
- iniciar un clúster de Kubernetes utilizando una terminal en línea



#### Introducción al cluster de Kubernetes

Kubernetes está diseñado como un grupo de computadoras altamente disponibles que están conectadas para funcionar como una sola unidad. Esta abstracción nos permite implementar aplicaciones sin pensar en qué máquinas específicas necesitan ejecutar. Para utilizar este nuevo modelo de implementación, las aplicaciones deben empaquetarse de manera que las desacople de los hosts individuales: deben estar en contenedor. Esto es diferente en comparación con la forma en que se implementaron las aplicaciones en el pasado, cuando se instalaron directamente en máquinas específicas como paquetes profundamente integrados en el host. La función de Kubernetes es automatizar la distribución (programación) de contenedores de aplicaciones en un clúster de una manera eficiente. Kubernetes es una plataforma de código abierto y está lista para producción.

Un clúster de Kubernetes se compone de 2 tipos de recursos:

- Master está coordinando el clúster
- Los nodos son donde ejecutamos aplicaciones



![Diagramde Cluster][img-clusterdiagram]



#### Cluster up and running

Check that it is properly installed, by running the minikube version command:

```
$ minikube version
minikube version: v0.17.1-katacoda
```


Start the cluster, by running the minikube start command:

```
$ minikube start
Starting local Kubernetes cluster...
```

Great! You now have a running Kubernetes cluster in your online terminal. Minikube started a virtual machine for you, and a Kubernetes cluster is now running in that VM.



#### Cluster version

We’re just going to look at some cluster information. To check if kubectl is installed you can run the kubectl version command:

```
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"8", GitVersion:"v1.8.0", GitCommit:"6e937839ac04a38cac63e6a7a306c5d035fe7b0a", GitTreeState:"clean", BuildDate:"2017-09-28T22:57:57Z", GoVersion:"go1.8.3", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"5", GitVersion:"v1.5.2", GitCommit:"08e099554f3c31f6e6f07b448ab3ed78d0520507", GitTreeState:"clean", BuildDate:"1970-01-01T00:00:00Z", GoVersion:"go1.7.1", Compiler:"gc", Platform:"linux/amd64"}
```

The client version is the kubectl version; the server version is the Kubernetes version installed on the master. 



#### Cluster details

```
$ kubectl cluster-info

Kubernetes master is running at http://host01:8080
heapster is running at http://host01:8080/api/v1/namespaces/kube-system/services/heapster/proxy
kubernetes-dashboard is running at http://host01:8080/api/v1/namespaces/kube-system/serv
ices/kubernetes-dashboard/proxy
monitoring-grafana is running at http://host01:8080/api/v1/namespaces/kube-system/servic
es/monitoring-grafana/proxymonitoring-influxdb is running at http://host01:8080/api/v1/namespaces/kube-system/services/monitoring-influxdb/proxy
```

To view the nodes in the cluster, run the kubectl get nodes command:

```
$ kubectl get nodes

NAME      STATUS    ROLES     AGE       VERSION
host01    Ready     <none>    5m        v1.5.2
```

This command shows all nodes that can be used to host our applications. Now we have only one node, and we can see that it’s status is ready (it is ready to accept applications for deployment).



## Módulo 2 - Implementar una aplicación

***Visión general del módulo***

- Obtenga información sobre implementaciones de aplicaciones
- Implemente su primera aplicación en Kubernetes con Kubectl


#### Su primera implementación de la aplicación

Una vez que tiene un clúster Kubernetes en ejecución, el siguiente paso es implementar sus aplicaciones en contenedores encima. En Kubernetes, un ***Despliegue*** es responsable de crear y actualizar instancias de su aplicación. El maestro programará instancias de aplicaciones creadas por la implementación en Nodos.

Después de crear las instancias de la aplicación, un ***Controlador*** de implementación las observa continuamente y reemplaza una instancia si el nodo que lo hospeda se desactiva o si se elimina. ***Esto proporciona un mecanismo de autocuración para abordar la falla de la máquina y el mantenimiento de la máquina.***

En un mundo de preorquestación, los scripts de instalación solían usarse para iniciar aplicaciones, pero no permitían la recuperación de fallas de la máquina. Al crear instancias de aplicaciones y mantenerlas ejecutándose en nodos, las implementaciones proporcionan un enfoque fundamentalmente diferente para la administración de aplicaciones.

![Proceso de despliegue][img-deployprocesses]

La imagen del contenedor de Implementación y el recuento de réplicas se definen en el momento de la creación, pero se pueden cambiar después de la creación. La actualización de implementaciones se tratará en los módulos 5 y 6 .

Usaremos la interfaz de línea de comandos de Kubernetes Kubectl para crear y administrar un Despliegue. Kubectl interactúa con el clúster a través de la API. Durante este bootcamp, aprenderá los comandos Kubectl más comunes necesarios para ejecutar sus aplicaciones en un clúster de Kubernetes.

Las aplicaciones deben estar empaquetadas en uno de los formatos de contenedor admitidos para ser implementadas en Kubernetes

En este campo de entrenamiento usaremos una aplicación NodeJS empaquetada en un contenedor Docker. El código fuente y el archivo Dockerf están disponibles en [url to github repo].

Ahora que ya sabe lo que son las implementaciones, ¡vamos al tutorial en línea y desplegamos nuestra primera aplicación!




#### kubectl basics

Like minikube, kubectl comes installed in the online terminal. Type kubectl in the terminal to see its usage. The common format of a kubectl command is: kubectl action resource This performs the specified action (like create, describe) on the specified resource (like node, container). You can use --help after the command to get additional info about possible parameters (kubectl get nodes --help).

Check that kubectl is configured to talk to your cluster, by running the kubectl version command:

```
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"8", GitVersion:"v1.8.0", GitCommit:"6e937839ac04a38cac63e6a7a306c5d035fe7b0a", GitTreeState:"clean", BuildDate:"2017-09-28T22:57:57Z", GoVersion:"go1.8.3", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"5", GitVersion:"v1.5.2", GitCommit:"08e099554f3c31f6e6f07b448ab3ed78d0520507", GitTreeState:"clean", BuildDate:"1970-01-01T00:00:00Z", GoVersion:"go1.7.1", Compiler:"gc", Platform:"linux/amd64"}
```

OK, kubectl is installed and you can see both the client and the server versions: 1.5.

To view the nodes in the cluster, run the kubectl get nodes command:

```
$ kubectl get nodes

NAME      STATUS    ROLES     AGE       VERSION
host01    Ready     <none>    4m        v1.5.2
```



#### Deploy our app

Let’s run our first app on Kubernetes with the kubectl run command. The run command creates a new deployment. We need to provide the deployment name and app image location (include the full repository url for images hosted outside Docker hub). We want to run the app on a specific port so we add the --port parameter:

```kubectl run kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1 --port=8080```

Great! You just deployed your first application by creating a deployment. This performed a few things for you:

- searched for a suitable node where an instance of the application could be run (we have only 1 available node)
- scheduled the application to run on that Node
- configured the cluster to reschedule the instance on a new Node when needed


To list your deployments use the `get deployments` command:

```
$ kubectl get deployments

NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1         1         1            1           1m
```

We see that there is 1 deployment running a single instance of your app. The instance is running inside a Docker container on your node.



#### View our app

Pods that are running inside Kubernetes are running on a private, isolated network. By default they are visible from other pods and services within the same kubernetes cluster, but not outside that network. When we use kubectl, we're interacting through an API endpoint to communicate with our application.

We will cover other options on how to expose your application outside the kubernetes cluster in Module 4.

The kubectl command can create a proxy that will forward communications into the cluster-wide, private network. The proxy can be terminated by pressing control-C and won't show any output while its running.

We will open a second terminal window to run the proxy.

```
$ kubectl proxy

Starting to serve on 127.0.0.1:8001
```


Ahora tenemos una conexión entre nuestro servidor (el terminal en línea) y el clúster de Kubernetes. El proxy permite el acceso directo a la API desde estos terminales.

Puede ver todas las API alojadas a través del punto final del proxy, ahora disponible en http: // localhost: 8001. Por ejemplo, podemos consultar la versión directamente a través de la API usando el comando curl:

`curl http://localhost:8001/version`

```
$ curl http://localhost:8001/version
{
  "major": "1",
  "minor": "5",
  "gitVersion": "v1.5.2",
  "gitCommit": "08e099554f3c31f6e6f07b448ab3ed78d0520507",
  "gitTreeState": "clean",
  "buildDate": "1970-01-01T00:00:00Z",
  "goVersion": "go1.7.1",
  "compiler": "gc",
  "platform": "linux/amd64"
}
$
```


El servidor API creará automáticamente un punto final para cada pod, en función del nombre del pod, al que también se puede acceder a través del proxy.

Primero necesitamos obtener el nombre del Pod y almacenaremos en la variable de entorno POD_NAME:

```
export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
```

```
echo Name of the Pod: $POD_NAME: kubernetes-bootcamp-390780338-lbpf1
```

```
$ kubectl get pods

NAME                                  READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-390780338-lbpf1   1/1       Running   0          7m
```


Ahora podemos hacer una solicitud HTTP a la aplicación que se ejecuta en ese pod:

```
curl http://localhost:8001/api/v1/proxy/namespaces/default/pods/$POD_NAME/

Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-390780338-339bx | v=1
```

La url es la ruta hacia la API del Pod.

Nota: Verifique la parte superior de la terminal. El proxy se ejecutó en una nueva pestaña (Terminal 2) y los comandos recientes se ejecutaron en la pestaña original (Terminal 1). El proxy aún se ejecuta en la segunda pestaña, y esto permitió que nuestro comando curl funcione usando localhost: 8001.




## Modulo 3 -Explora tu aplicación

- Aprende sobre los Pods de Kubernetes
- Aprende sobre los Nodos de Kubernetes
- Solucionar problemas de aplicaciones implementadas


#### Pods

En el Módulo 2 , el Despliegue creó un Podque alojó su instancia de aplicación. Un Pod es un grupo de uno o más contenedores de aplicaciones (como Docker o rkt) que incluye almacenamiento compartido (volúmenes), una única dirección IP del clúster e información sobre cómo ejecutarlos (como la versión de imagen del contenedor o puertos específicos). Los contenedores dentro de un Pod comparten una dirección IP y espacio en el puerto. Los contenedores del mismo Pod siempre se ubican y reprograman conjuntamente, y se ejecutan en un contexto compartido en el mismo nodo. Un Pod modela un "host lógico" específico de la aplicación y contiene uno o más contenedores de aplicaciones que están relativamente estrechamente acoplados. Un ejemplo de contenedor que cabría en el mismo Pod con nuestra aplicación NodeJS sería un contenedor lateral que alimenta los datos publicados por el servidor web. En un mundo previo al contenedor, se habrían ejecutado en la misma máquina física o virtual.

Los pods están vinculados al nodo donde se implementan y permanecen allí hasta la terminación (de acuerdo con la política de reinicio) o la eliminación. En caso de una falla de Nodo, se desplegarán nuevos Pods idénticos en otros Nodos disponibles. El Pod es la unidad de despliegue atómica en la plataforma de Kubernetes. Cuando desencadenamos un Despliegue en Kubernetes, creará Pods con contenedores dentro de ellos, no contenedores directamente.




[img-clusterdiagram]: https://kubernetesbootcamp.github.io/kubernetes-bootcamp/public/images/module_01_cluster.svg "Diagrama de cluster"

[img-deployprocesses]: https://kubernetesbootcamp.github.io/kubernetes-bootcamp/public/images/module_02_first_app.svg