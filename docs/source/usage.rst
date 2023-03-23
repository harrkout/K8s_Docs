Kubernetes guide
================

--------------

Prerequisites
~~~~~~~~~~~~~

**Master & Worker Nodes**

``docker.io``-> Container runtime ``kubelet`` -> Daemon running on
systemd. CRUD containers on Pods. ``kubeadm`` -> Performs the necessary
actions to get a minimum viable cluster up and running. ``kubectl`` ->
CLI againts K8s clusters, e.g. deploy applications, inspect and manage
cluster resources, and view logs.

Optional packages
~~~~~~~~~~~~~~~~~

``https transport``

``curl``

--------------

Getting started
---------------

Creating a cluster with minikube on host machine

   Install minikube to set a local K8s cluster. Not OS-specific.

``minikube start``

``kubectl version --output=yaml`` ``kubectl cluster-info``

View nodes in the cluster

``kubectl get nodes``

Deployment
----------

``kubectl create deployment <name> --image<image-name-location>``
``kubectl create deployment <name> --image<image-name-location>`` >
e.g. => kubectl create deployment kubernetes-bootcamp
–image=gcr.io/google-samples/kubernetes-bootcamp:v1

List your deployments:

``kubectl get deplyments``

   Pods that are running inside Kubernetes are running on a private,
   isolated network. By default they are visible from other pods and
   services within the same kubernetes cluster, but not outside that
   network. When we use kubectl, we're interacting through an API
   endpoint to communicate with or application.

In another terminal, run:

.. code:: console

   echo -e "\n\n\n\e[92mStarting Proxy. After starting it will not output a response.
   Please click the first Terminal Tab\n"; 
   kubectl proxy

In yet another terminal, run:

.. code:: console

   curl https://localhost:8001/version

..

   The API server will automatically create an endpoint for each pod,
   based on the pod name, that is also accessible through the proxy.
   First we need to get the Pod name, and we'll store in the environment
   variable POD_NAME:

.. code:: console

   export POD_NAME=$(kubectl get pods -o go-template --template
    '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')

You can access the Pod through the API by running:

.. code:: console

   curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/ 

--------------

Pod examination and configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Pod status can be seen via the describe command of kubectl.

.. code:: console

   kubectl describe pod <pod-name>

Access to the contairer inside the Pod can be granted via the following
command, which iterrates an interactive terminal.

.. code:: console

   kubectl exec -it <pod-name> -- bin/bash

--------------

The Deployment commands are the following:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: console

   kubectl create deployment [name]
   kubectl edit deployment [name]
   kubectl delete deployment [name]

--------------

Status of different K8s components
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: console

   kubectl get nodes|pod|services|replicaset|deployment

--------------

Debugging pods
^^^^^^^^^^^^^^

.. code:: console

   kubectl logs [pod name]
   kubectl exec -it [pod-name] -- bin/bash

--------------

Creating a custom configuration file
------------------------------------

Configuration files in K8s are of .yaml file format. After a Pod,
Container and Deployment are created, a config file can be
created/edited.

Creating a .yaml configuration file:

.. code:: console

   touch nginx-deployment.yaml
   nvim nginx-deployment.yaml

..

   Nginx is a local web server that provides load balancing, allong with
   HTTP cache and reverse proxy.

   From the .yaml configuration file that will be created below, the
   nginx local server will be deployed inside a containerized
   environment.

Inside the .yaml file, a strict and specific syntax must be followed.

**Indentation must be strictly followed, otherwise it leads to errors.**

.. code:: console

   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-deployment
     labels:
       app: nginx
   spec:       ##specification for the deployment
     replicas: 2
     selector:
       matchLabels:
         app: nginx
     template:
       metadata:
         labels:
           app: nginx
       spec:   ##specification for the pods
         containers:
         - name: nginx
           image: nginx
           ports:
           - containerPort: 80

--------------

User configuration files for CRUD.
''''''''''''''''''''''''''''''''''

After the config file has been created, it can be applied via the
following command:

.. code:: console

   kubectl apply -f nginx-deployment.yaml

If we run the following command, we can see the new deployment is ready
and running:

.. code:: console

   kubectl get pod

Similarly for the deployment:

.. code:: console

   kubectl get deployment

--------------

Layers of Abstraction:
~~~~~~~~~~~~~~~~~~~~~~

Deployment -> ReplicaSet -> Pod -> Container
''''''''''''''''''''''''''''''''''''''''''''

--------------

YAML Configuration File
-----------------------

**Strict Syntax Indentation!!**
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

   For autogenerating config files, K8s gets the status from the etch,
   which hold the current status of any K8s component!

The basic idea is that inside a .yaml configuration file exist other
configuration files as ``metadata`` and ``spec`` sections.

``Pods`` should have their own configuration inside of the
``Deployments`` configuration file. All ``Pods`` will be defined.

Inside the ``metadata`` of each ``pod``, exist the ``spec`` section. The
``spec`` section covers the name of the ``container``, the ``image``
running inside the ``container``, along with the ``containerPort``
inside the private network.

The connection between ``Services`` and ``Deployments`` is established
with ``Labels`` and ``Selectors``.

Specificlly, the ``metadata`` part contains the ``labels``, and the
``spec`` part contains ``Selectors``.

This way, the ``Deployment`` knows with ``Pods`` belong to specific
applications.

The ``Deployment`` has its own label, which will be used by the
``Service`` selector which makes a connection between the ``Service``
and the ``Deployment``.

Ports
^^^^^

Both ``Service`` and ``Deployment`` need to have ``Ports`` defined. That
way, the DB Service knows with which port to communicate with the nginx
Service, and to which ``Pod`` it should forward the request, but also
which ``Pods`` are listening.

**Some examples:**

.. raw:: html

   <table>

.. raw:: html

   <tr>

.. raw:: html

   <th>

nginx-deployment.yaml

.. raw:: html

   </th>

.. raw:: html

   <th>

nginx-service.yaml

.. raw:: html

   </th>

.. raw:: html

   </tr>

.. raw:: html

   <tr>

.. raw:: html

   <td>

.. raw:: html

   <pre>
     apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-deployment
     labels:
       app: nginx
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: nginx
     template:
       metadata:
         labels:
           app: nginx
       spec:
         containers:
         - name: nginx
           image: nginx:1.16
           ports:
           - containerPort: 8080
   </pre>

.. raw:: html

   </td>

.. raw:: html

   <td>

.. code:: json

   apiVersion: v1
   kind: Service
   metadata:
     name: nginx-service
   spec:
     selector:
       app: nginx
     ports:
       - protocol: TCP
         port: 80
         targetPort: 8080
    

.. raw:: html

   </td>

.. raw:: html

   </tr>

.. raw:: html

   </table>

After the configuration files are created, we can apply them to both
deployment and service.

``kubectl apply -f nginx-deployment.yaml``

``kubectl apply -f nginx-service.yaml``

Now we can see that 2 replicas are running, as it was defined in the
config file of the ``nginx-deployment.yaml``

``kubectl get pod``

And the service we created from the ``nginx-service.yaml`` is up and
running.

``kubectl get service``

--------------

Now we can get the information of the auto-generated config file of the
``nginx-service`` by running:

``kubectl describe service nginx-service``

Inside which description, we can find the ``Endpoints``, which describe
the IP-Addresses of the Pods, along with the port they’re listening.

We can check if the ports of the ``Pods`` are correct by running:

``kubectl get pod -o wide``

--------------

Finally, let’s check the status, in .yaml format, that K8s automatically
generates, and save it in a file: The status info resides in the
``etcd``, which stores the the status of the whole cluster, including
every component.

``kubectl get deployment nginx-deployment -o yaml > nginx-deployment-result.yaml``

   .. raw:: html

      <tr>

   .. raw:: html

      <th>

   nginx-deployment-result.yaml

   .. raw:: html

      </th>

   .. raw:: html

      </tr>

   .. raw:: html

      <tr>

   .. raw:: html

      <td>

.. code:: json

   apiVersion: apps/v1
   kind: Deployment
   metadata:
     annotations:
       deployment.kubernetes.io/revision: "1"
       kubectl.kubernetes.io/last-applied-configuration: |
         {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"nginx"},"name":"nginx-deployment","namespace":"default"},"spec":{"replicas":2,"selector":{"matchLabels":{"app":"nginx"}},"template":{"metadata":{"labels":{"app":"nginx"}},"spec":{"containers":[{"image":"nginx:1.16","name":"nginx","ports":[{"containerPort":8080}]}]}}}}
     creationTimestamp: "2023-03-23T10:54:56Z"
     generation: 1
     labels:
       app: nginx
     name: nginx-deployment
     namespace: default
     resourceVersion: "96574"
     selfLink: /apis/apps/v1/namespaces/default/deployments/nginx-deployment
     uid: e1075fa3-6468-43d0-83c0-63fede0dae51
   spec:
     progressDeadlineSeconds: 600
     replicas: 2
     revisionHistoryLimit: 10
     selector:
       matchLabels:
         app: nginx
     strategy:
       rollingUpdate:
         maxSurge: 25%
         maxUnavailable: 25%
       type: RollingUpdate
     template:
       metadata:
         creationTimestamp: null
         labels:
           app: nginx
       spec:
         containers:
         - image: nginx:1.16
           imagePullPolicy: IfNotPresent
           name: nginx
           ports:
           - containerPort: 8080
             protocol: TCP
           resources: {}
           terminationMessagePath: /dev/termination-log
           terminationMessagePolicy: File
         dnsPolicy: ClusterFirst
         restartPolicy: Always
         schedulerName: default-scheduler
         securityContext: {}
         terminationGracePeriodSeconds: 30
   status:
     availableReplicas: 2
     conditions:
     - lastTransitionTime: "2023-03-23T10:54:59Z"
       lastUpdateTime: "2023-03-23T10:54:59Z"
       message: Deployment has minimum availability.
       reason: MinimumReplicasAvailable
       status: "True"
       type: Available
     - lastTransitionTime: "2023-03-23T10:54:56Z"
       lastUpdateTime: "2023-03-23T10:54:59Z"
       message: ReplicaSet "nginx-deployment-7d64f4b574" has successfully progressed.
       reason: NewReplicaSetAvailable
       status: "True"
       type: Progressing
     observedGeneration: 1
     readyReplicas: 2
     replicas: 2
     updatedReplicas: 2

**We can delete the ``Deployment`` and ``Service`` by deleting the
configuration files.**

``kubectl delete -f nginx-deployment.yaml``
``kubectl delete -f nginx-service.yaml``

--------------

Complete Application Setup
--------------------------

Implementation of simple web application using ``mongo-express`` &
``mongoDB``.

First, we’re going to create a ``mongoDB`` ``Pod``, and to talk to the
``Pod`` we’re going to need a service. We’re going to create an Internal
Service, meaning that no external requests are allowed to the pod, only
components in the same cluster are able to talk to it.

Then we’ll create a ``mongo-express`` ``Deployment``

We’ll create a ``Deployment.yaml`` for the ``mongo-express`` deployment,
which will be provided with environmental variables, that will allow it
to connect to the ``mongoDB``.

The ``mongoDB`` will consist of the following:

-  ConfigMap -> DB URL
-  Secret -> DB User, DB Pwd

So the Request Flow will look like the following:

   The request comes from the browser, it goes through the
   ``mongo-express external Service``, which will forward it to the
   ``mongo-express`` ``Pod``. The ``Pod`` then will connect to the
   ``mongoDB`` Internal Service, which will forward it to the
   ``mongoDB`` ``Pod``, where it will authenticate the request by using
   the credentials of the ``Secret`` module of the ``mongoDB``.

-  Start ``minikube`` if using a local cluster instance in your host
   machine.

-  Run ``kubectl get all`` to view all the components inside the
   cluster.

Step 1:
'''''''

Create the ``mongoDB`` Deployment.

.. raw:: html

   <table>

.. raw:: html

   <tr>

.. raw:: html

   <th>

mongo.yaml

.. raw:: html

   </th>

.. raw:: html

   <th>

mongodb-secret.yaml

.. raw:: html

   </th>

.. raw:: html

   </tr>

.. raw:: html

   <tr>

.. raw:: html

   <td>

.. raw:: html

   <pre>
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: mongodb-deployment
     labels:
       app: mongodb
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: mongodb
     template:
       metadata:
         labels:
           app: mongodb
       spec:
         containers:
         - name: mongodb
           image: mongo
           ports:
           - containerPort: 27017
           env:
           - name: MONGO_INITDB_ROOT_USERNAME
             valueFrom:
               secretKeyRef:
                 name: mongodb-secret
                 key: mongo-root-username
           - name: MONGO_INITDB_ROOT_PASSWORD
             valueFrom: 
               secretKeyRef:
                 name: mongodb-secret
                 key: mongo-root-password
     ---
     apiVersion: v1
     kind: Service
     metadata:
       name: mongodb-service
     spec:
       selector:
         app: mongodb
       ports:
         - protocol: TCP
           port: 27017
           targetPort: 27017

   </pre>

.. raw:: html

   </td>

.. raw:: html

   <td>

::

   apiVersion: v1
   kind: Secret
   metadata:
       name: mongodb-secret
   type: Opaque
   data:
       mongo-root-username: dXNlcm5hbWU=    #generated as base64 value with command 
       mongo-root-password: cGFzc3dvcmQ=    # `echo -n 'username/password' | base64

.. raw:: html

   </td>

.. raw:: html

   </tr>

.. raw:: html

   </table>

Then we can apply the secret with ``kubectl``

``kubectl apply -f mongodb-secret.yaml``

Check the ``secret`` status:

``kubectl get secret``

Create the deployment

``kubectl apply -f mongo.yaml``

Check the pod status

``kubectl get pod``

   If it takes a bit for the ``pod`` to be created, you can run
   ``kubectl get pod --watch`` to have live feedback.

+---+
| # |
| S |
| t |
| e |
| p |
| 2 |
| : |
| C |
| r |
| e |
| a |
| t |
| e |
| a |
| n |
| i |
| n |
| t |
| e |
| r |
| n |
| a |
| l |
| s |
| e |
| r |
| v |
| i |
| c |
| e |
| s |
| o |
| t |
| h |
| a |
| t |
| o |
| t |
| h |
| e |
| r |
| ` |
| ` |
| p |
| o |
| d |
| s |
| ` |
| ` |
| c |
| a |
| n |
| t |
| a |
| l |
| k |
| t |
| o |
| t |
| h |
| e |
| ` |
| ` |
| m |
| o |
| n |
| g |
| o |
| d |
| b |
| ` |
| ` |
+===+
| a |
| p |
| i |
| V |
| e |
| r |
| s |
| i |
| o |
| n |
| : |
| v |
| 1 |
| k |
| i |
| n |
| d |
| : |
| S |
| e |
| r |
| v |
| i |
| c |
| e |
| m |
| e |
| t |
| a |
| d |
| a |
| t |
| a |
| : |
| n |
| a |
| m |
| e |
| : |
| m |
| o |
| n |
| g |
| o |
| d |
| b |
| - |
| s |
| e |
| r |
| v |
| i |
| c |
| e |
| s |
| p |
| e |
| c |
| : |
| s |
| e |
| l |
| e |
| c |
| t |
| o |
| r |
| : |
| a |
| p |
| p |
| : |
| m |
| o |
| n |
| g |
| o |
| d |
| b |
| p |
| o |
| r |
| t |
| s |
| : |
| - |
| p |
| r |
| o |
| t |
| o |
| c |
| o |
| l |
| : |
| T |
| C |
| P |
| p |
| o |
| r |
| t |
| : |
| 2 |
| 7 |
| 0 |
| 1 |
| 7 |
| t |
| a |
| r |
| g |
| e |
| t |
| P |
| o |
| r |
| t |
| : |
| 2 |
| 7 |
| 0 |
| 1 |
| 7 |
+---+
| p |
| r |
| e |
| > |
| t |
| d |
| > |
| t |
| r |
| > |
| t |
| a |
| b |
| l |
| e |
| > |
+---+
| r |
| / |
| > |
+---+
| # |
| # |
| A |
| p |
| p |
| l |
| y |
| t |
| h |
| e |
| C |
| o |
| n |
| f |
| i |
| g |
| M |
| a |
| p |
| : |
+---+
| u |
| b |
| e |
| c |
| t |
| l |
| a |
| p |
| p |
| l |
| y |
| - |
| f |
| m |
| o |
| n |
| g |
| o |
| - |
| c |
| o |
| n |
| f |
| i |
| g |
| m |
| a |
| p |
| . |
| y |
| a |
| m |
| l |
| \ |
| ` |
+---+
| # |
| # |
| A |
| p |
| p |
| l |
| y |
| t |
| h |
| e |
| M |
| o |
| n |
| g |
| o |
| - |
| E |
| x |
| p |
| r |
| e |
| s |
| s |
| : |
+---+
| u |
| b |
| e |
| c |
| t |
| l |
| a |
| p |
| p |
| l |
| y |
| - |
| f |
| m |
| o |
| n |
| g |
| o |
| - |
| e |
| x |
| p |
| r |
| e |
| s |
| s |
| . |
| y |
| a |
| m |
| l |
| \ |
| ` |
+---+
| c |
| a |
| n |
| s |
| e |
| e |
| t |
| h |
| e |
| l |
| o |
| g |
| s |
| f |
| o |
| r |
| f |
| u |
| r |
| t |
| h |
| e |
| r |
| i |
| n |
| f |
| o |
| r |
| m |
| a |
| t |
| i |
| o |
| n |
| a |
| n |
| d |
| c |
| o |
| n |
| f |
| i |
| r |
| m |
| a |
| t |
| i |
| o |
| n |
| t |
| h |
| a |
| t |
| e |
| v |
| e |
| r |
| y |
| t |
| h |
| i |
| n |
| g |
| i |
| s |
| g |
| o |
| i |
| n |
| g |
| s |
| m |
| o |
| o |
| t |
| h |
| l |
| y |
| : |
+---+
| u |
| b |
| e |
| c |
| t |
| l |
| l |
| o |
| g |
| s |
| m |
| o |
| n |
| g |
| o |
| - |
| e |
| x |
| p |
| r |
| e |
| s |
| s |
| - |
| 5 |
| b |
| f |
| 4 |
| b |
| 5 |
| 6 |
| f |
| 4 |
| 7 |
| - |
| 5 |
| n |
| 9 |
| v |
| q |
| \ |
| ` |
| h |
| a |
| n |
| g |
| e |
| t |
| h |
| e |
| n |
| a |
| m |
| e |
| o |
| f |
| t |
| h |
| e |
| m |
| o |
| n |
| g |
| o |
| - |
| e |
| x |
| p |
| r |
| e |
| s |
| s |
| w |
| i |
| t |
| h |
| t |
| h |
| e |
| n |
| a |
| m |
| e |
| i |
| f |
| t |
| h |
| e |
| i |
| n |
| s |
| t |
| a |
| n |
| c |
| e |
| i |
| n |
| y |
| o |
| u |
| r |
| m |
| a |
| c |
| h |
| i |
| n |
| e |
| . |
+---+
| a |
| b |
| l |
| e |
| > |
| r |
| > |
| h |
| > |
| M |
| o |
| n |
| g |
| o |
| - |
| e |
| x |
| p |
| r |
| e |
| s |
| s |
| l |
| o |
| g |
| s |
+---+
| l |
| c |
| o |
| m |
| e |
| t |
| o |
| m |
| o |
| n |
| g |
| o |
| - |
| e |
| x |
| p |
| r |
| e |
| s |
| s |
+---+

(node:7) [MONGODB DRIVER] Warning: Current Server Discovery and
Monitoring engine is deprecated, and will be removed in a future
version. To use the new Server Discover and Monitoring engine, pass
option { useUnifiedTopology: true } to the MongoClient constructor.
Mongo Express server listening at http://0.0.0.0:8081 Server is open to
allow connections from anyone (0.0.0.0) basicAuth credentials are
“admin:pass”, it is recommended you change this in your config.js!

.. raw:: html

   </pre>

.. raw:: html

   </td>

.. raw:: html

   </tr>

.. raw:: html

   </table>

Now that everything is running correctly, the last step is to create an
external service so that we can access the mongo-express from a browser.

--------------

Let’s create an external service for the mongo-express.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. raw:: html

   <table>

.. raw:: html

   <tr>

.. raw:: html

   <th>

mongo-express.yaml

.. raw:: html

   </th>

.. raw:: html

   </tr>

.. raw:: html

   <tr>

.. raw:: html

   <td>

.. raw:: html

   <pre>
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: mongo-express
     labels:
       app: mongo-express
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: mongo-express
     template:
       metadata:
         labels:
           app: mongo-express
       spec:
         containers:
         - name: mongo-express
           image: mongo-express
           ports:
           - containerPort: 8081
           env:
           - name: ME_CONFIG_MONGODB_ADMINUSERNAME
             valueFrom:
               secretKeyRef:
                 name: mongodb-secret
                 key: mongo-root-username
           - name: ME_CONFIG_MONGODB_ADMINPASSWORD
             valueFrom: 
               secretKeyRef:
                 name: mongodb-secret
                 key: mongo-root-password
           - name: ME_CONFIG_MONGODB_SERVER
             valueFrom: 
               configMapKeyRef:
                 name: mongodb-configmap
                 key: database_url
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: mongo-express-service
   spec:
     selector:
       app: mongo-express
     type: LoadBalancer  
     ports:
       - protocol: TCP
         port: 8081
         targetPort: 8081
         nodePort: 30000/pre>
   </pre>

.. raw:: html

   </td>

.. raw:: html

   </tr>

.. raw:: html

   </table>

Now, running the command:

``kubectl get service``

will give us all the information we need for the service that was
created. Most importantly, the Cluster-IP address, along with port and
the type of the service.

The External-IP address is not yet specified, so we need to assign to an
external service a public IP-Adress.

``minikube service mongo-express-service``

And as a result, a browser will open automatically to the Mongo Express
page.

To-Do Sections:
===============

-  K8s Namespaces
-  K8s Ingress
-  Helm Package Manager
-  Persisting Data in Volumes
-  Deploying Stateful Apps with StatefulSet
