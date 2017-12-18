# Kubernetes



## Módulo 1 Crear un clúster de Kubernetes

***Visión general del módulo***

- aprende qué es un cluster de Kubernetes
- aprende qué es minikube
- iniciar un clúster de Kubernetes utilizando una terminal en línea



#### Introducción al cluster de Kubernetes

Kubernetes está diseñado como un grupo de computadoras altamente disponibles que están conectadas para funcionar como una sola unidad. Esta abstracción nos permite implementar aplicaciones sin pensar en qué máquinas específicas necesitan ejecutar. 

Para utilizar este nuevo modelo de implementación, las aplicaciones deben empaquetarse de manera que las desacople de los hosts individuales: deben estar en contenedor. Esto es diferente en comparación con la forma en que se implementaron las aplicaciones en el pasado, cuando se instalaron directamente en máquinas específicas como paquetes profundamente integrados en el host. La función de Kubernetes es automatizar la distribución (programación) de contenedores de aplicaciones en un clúster de una manera eficiente. Kubernetes es una plataforma de código abierto y está lista para producción.

Un clúster de Kubernetes se compone de 2 tipos de recursos:

- Master está coordinando el clúster
- Los nodos son donde ejecutamos aplicaciones



![Diagramde Cluster][img-clusterdiagram]



#### Cluster funcionando

Verifique que esté instalado correctamente, ejecutando el comando de versión de minikube:

```
$ minikube version
minikube version: v0.17.1-katacoda
```


Inicie el clúster ejecutando el comando minikube start:

```
$ minikube start
Starting local Kubernetes cluster...
```

¡Estupendo! Ahora tiene un clúster Kubernetes en ejecución en su terminal en línea. Minikube inició una máquina virtual para usted, y ahora se está ejecutando un clúster de Kubernetes en esa máquina virtual.



#### Versión de clúster

Solo vamos a ver información del clúster. Para verificar si kubectl está instalado, puede ejecutar el comando de versión kubectl:

```
$ kubectl version

Client Version: version.Info{Major:"1", Minor:"8", GitVersion:"v1.8.0", GitCommit:"6e937839ac04a38cac63e6a7a306c5d035fe7b0a", GitTreeState:"clean", BuildDate:"2017-09-28T22:57:57Z", GoVersion:"go1.8.3", Compiler:"gc", Platform:"linux/amd64"}

Server Version: version.Info{Major:"1", Minor:"5", GitVersion:"v1.5.2", GitCommit:"08e099554f3c31f6e6f07b448ab3ed78d0520507", GitTreeState:"clean", BuildDate:"1970-01-01T00:00:00Z", GoVersion:"go1.7.1", Compiler:"gc", Platform:"linux/amd64"}
```

La versión del cliente es la versión de kubectl; la versión del servidor es la versión de Kubernetes instalada en el maestro.



#### Detalles del cluster

```
$ kubectl cluster-info

Kubernetes master is running at http://host01:8080
heapster is running at http://host01:8080/api/v1/namespaces/kube-system/services/heapster/proxy
kubernetes-dashboard is running at http://host01:8080/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy
monitoring-grafana is running at http://host01:8080/api/v1/namespaces/kube-system/services/monitoring-grafana/proxymonitoring-influxdb is running at http://host01:8080/api/v1/namespaces/kube-system/services/monitoring-influxdb/proxy
```

Para ver los nodos en el clúster, ejecute el comando kubectl get nodes:

```
$ kubectl get nodes

NAME      STATUS    ROLES     AGE       VERSION
host01    Ready     <none>    5m        v1.5.2
```

Este comando muestra todos los nodos que se pueden usar para alojar nuestras aplicaciones. Ahora solo tenemos un nodo, y podemos ver que su estado está listo (está listo para aceptar aplicaciones para la implementación).



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

Al igual que minikube, kubectl viene instalado en la terminal en línea. Escriba kubectl en la terminal para ver su uso. 

El formato común de un comando kubectl es: `kubectl recurso de acción`. Esto realiza la acción especificada (como crear, describir) en el recurso especificado (como nodo, contenedor). Puede usar --help after the command para obtener información adicional sobre los posibles parámetros (kubectl get nodes --help).

Check that kubectl is configured to talk to your cluster, by running the kubectl version command:

```
$ kubectl version

Client Version: version.Info{Major:"1", Minor:"8", GitVersion:"v1.8.0", GitCommit:"6e937839ac04a38cac63e6a7a306c5d035fe7b0a", GitTreeState:"clean", BuildDate:"2017-09-28T22:57:57Z", GoVersion:"go1.8.3", Compiler:"gc", Platform:"linux/amd64"}

Server Version: version.Info{Major:"1", Minor:"5", GitVersion:"v1.5.2", GitCommit:"08e099554f3c31f6e6f07b448ab3ed78d0520507", GitTreeState:"clean", BuildDate:"1970-01-01T00:00:00Z", GoVersion:"go1.7.1", Compiler:"gc", Platform:"linux/amd64"}
```

De acuerdo, kubectl está instalado y puede ver las versiones del cliente y del servidor: 1.5.

Para ver los nodos en el clúster, ejecute el comando kubectl get nodes:

```
$ kubectl get nodes

NAME      STATUS    ROLES     AGE       VERSION
host01    Ready     <none>    4m        v1.5.2
```



#### Deploy our app

Vamos a ejecutar nuestra primera aplicación en Kubernetes con el comando de ejecución kubectl. El comando de ejecución crea una nueva implementación. Necesitamos proporcionar el nombre del despliegue y la ubicación de la imagen de la aplicación (incluya la url del repositorio completo para las imágenes alojadas fuera del centro Docker). 

Queremos ejecutar la aplicación en un puerto específico, así que agregamos el parámetro de puerto:

```kubectl run kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1 --port=8080```

¡Estupendo! Acaba de implementar su primera aplicación creando una implementación. Esto realizó las siguientes tareas:

- busqué un nodo adecuado donde se pudiera ejecutar una instancia de la aplicación (solo tenemos 1 nodo disponible)
- programar la aplicación para que se ejecute en ese nodo
- configurado el clúster para reprogramar la instancia en un nuevo nodo cuando sea necesario


Para enumerar sus implementaciones, use el comando `get deployments`:

```
$ kubectl get deployments

NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1         1         1            1           1m
```

Vemos que hay 1 implementación que ejecuta una sola instancia de su aplicación. La instancia se está ejecutando dentro de un contenedor Docker en su nodo.



#### Ver nuestra aplicación

Los pods que se ejecutan dentro de Kubernetes se ejecutan en una red privada y aislada. De forma predeterminada, son visibles desde otros pods y servicios dentro del mismo clúster de kubernetes, pero no desde esa red. Cuando usamos kubectl, estamos interactuando a través de un enpoint API para comunicarnos con nuestra aplicación.

Cubriremos otras opciones sobre cómo exponer su aplicación fuera del clúster de kubernetes en el Módulo 4.

El comando kubectl puede crear un proxy que reenviará las comunicaciones a la red privada de todo el clúster. El proxy puede terminarse presionando control-C y no mostrará ningún resultado mientras se está ejecutando.

Abriremos una segunda ventana de terminal para ejecutar el proxy.

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


El servidor API creará automáticamente un endpoint para cada pod, en función del nombre del pod, al que también se puede acceder a través del proxy.

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

En el Módulo 2 , el Despliegue creó un Pod donde alojó su instancia de aplicación. 

Un Pod es un grupo de uno o más contenedores de aplicaciones (como Docker o rkt) que incluye almacenamiento compartido (volúmenes), una única dirección IP del clúster e información sobre cómo ejecutarlos (como la versión de imagen del contenedor o puertos específicos). 

Los contenedores dentro de un Pod comparten una dirección IP y espacio en el puerto. 

Los contenedores del mismo Pod siempre se ubican y reprograman conjuntamente, y se ejecutan en un contexto compartido en el mismo nodo. 

Un Pod modela un "host lógico" específico de la aplicación y contiene uno o más contenedores de aplicaciones que están relativamente estrechamente acoplados. Un ejemplo de contenedor que cabría en el mismo Pod con nuestra aplicación NodeJS sería un contenedor lateral que alimenta los datos publicados por el servidor web. En un mundo previo al contenedor, se habrían ejecutado en la misma máquina física o virtual.

Los pods están vinculados al nodo donde se implementan y permanecen allí hasta la terminación (de acuerdo con la política de reinicio) o la eliminación. En caso de una falla de Nodo, se desplegarán nuevos Pods idénticos en otros Nodos disponibles. El Pod es la unidad de despliegue atómica en la plataforma de Kubernetes. Cuando desencadenamos un Despliegue en Kubernetes, creará Pods con contenedores dentro de ellos, no contenedores directamente.


Visión general de los pods

![][img-podsoverview]



#### Nodos

Los Pods siempre se ejecutan en nodos. Un Nodo es una máquina de trabajo en Kubernetes y puede ser una VM o una máquina física, dependiendo del clúster. Cada Nodo ejecuta Pods y es administrado por el Maestro. 

En un Nodo puedes tener múltiples pods. La programación de los Pods se realiza automáticamente por el Maestro y esto tiene en cuenta los recursos disponibles en los Nodos.

Cada nodo de Kubernetes ejecuta al menos:

- Un contenedor (como Docker, rkt) que se encargará de sacar todos sus contenedores de un registro.
- Kubelet, que actúa como un puente entre el Maestro de Kubernetes y los Nodos; administra los Pods y los contenedores que se ejecutan en una máquina.

Visión general del nodo

![][img-nodooverview]



#### Solución de problemas con kubectl

En el Módulo 2 presentamos la herramienta kubelet cli. Lo usaremos en este módulo para obtener información sobre las aplicaciones implementadas y sus entornos. Las operaciones más comunes se pueden hacer con los siguientes comandos kubectl:

- ***kubectl get*** - lista recursos
- ***kubectl describe*** - muestra información detallada sobre un recurso
- ***kubectl logs*** - imprime los logs de un contenedor en un pod
- ***kubectl exec*** - ejecuta un comando en un contenedor en un pod

Esos comandos lo ayudarán a ver cuándo se implementaron las aplicaciones, cuál es su estado actual, dónde se están ejecutando y cuál es su configuración.

Ahora que sabemos más sobre nuestros componentes de clúster y la línea de comandos, exploremos nuestra aplicación.











[img-clusterdiagram]: https://kubernetesbootcamp.github.io/kubernetes-bootcamp/public/images/module_01_cluster.svg "Diagrama de cluster"

[img-deployprocesses]: https://kubernetesbootcamp.github.io/kubernetes-bootcamp/public/images/module_02_first_app.svg

[img-podsoverview]: https://kubernetesbootcamp.github.io/kubernetes-bootcamp/public/images/module_03_pods.svg

[img-nodooverview]: https://kubernetesbootcamp.github.io/kubernetes-bootcamp/public/images/module_03_nodes.svg