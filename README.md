# Kubernetes general used commands and shortly definings from **Kubernetes In Action** 

**port-forwarding** // The port forwarder is running and you can now connect to your pod through the local port.

>  **$ kubectl port-forward kubia-manual 8888:8080**


To see your **pod’s log** (more precisely, the container’s log) you run the following 
command on your local machine (no need to ssh anywhere)

> **$ kubectl logs kubia-manual**                                                             
  Kubia server starting...

To create the pod from your YAML file
> **$ kubectl create -f kubia-manual.yaml**                                                            
pod "kubia-manual" created

Retrieving the whole definition of a running pod
> **$ kubectl get pod kubia-manual -o yaml**

***LABELS***                                                                                             
The **kubectl get pods**  command  doesn’t  list  any  labels  by  default,  but  you  can  seet hem by using 
the **--show-labels** switch:
> **$ kubectl get pods --show-labels**                                                                      
   NAME------------READY--STATUS---RESTARTS--AGE--LABELS                                                      
   kubia-manual----1/1----Running--0---------16m--<none>                                                        
   kubia-manual-v2-1/1----Running--0---------2m---creat_method=manual,env=prod                                   
   kubia-zxzij-----1/1----Running--0---------1d---run=kubia                                                 
   
Instead of listing all labels, if you’re only **interested in certain labels**, you can specify them 
with the **-L** switch and have each displayed in its own column                                       
> **$ kubectl get pods **-L** creation_method,env**                                                              
   NAME------------READY---STATUS----RESTARTS---AGE---CREATION_METHOD---ENV                                                    
   kubia-manual----1/1-----Running---0----------16m---<none>------------<none>                                                 
   kubia-manual-v2-1/1-----Running---0----------2m----manual------------prod                                                     
   kubia-zxzij-----1/1-----Running---0----------1d----<none>------------<none>                                     
   
Labels can also be **added** to and **modified** on existing pods.
> **$ kubectl label pod kubia-manual creation_method=manual**                                                         
pod "kubia-manual" labeled                                                                                  

You need to use the --overwrite option when changing existing labels.                                     
> **$ kubectl label pod kubia-manual-v2 env=debug --overwrite**                                                      
pod "kubia-manual-v2" labeledTwo labels are attached to the pod.

Listing pods using a label selector                                                                             
--query to specific label--
> **$ kubectl get pods -l creation_method=manual**                                                                                                                                                              
   NAME--------------READY----STATUS-----RESTARTS---AGE                                                                          
   kubia-manual------1/1------Running----0----------51                                                                                
   mkubia-manual-v2--1/1------Running----0----------37m

   
Using a label selector to schedule a pod to a specific node:

    apiVersion: v1
    kind: Pod
    metadata: 
     name: kubia-gpu
    spec:
     nodeSelector:  //nodeSelector tells Kubernetes to deploy this pod only to nodes containing the gpu=true label.
      gpu: "true"
    containers:
      - image: luksa/kubia
      name: kubia
  
  ***NAMESPACES***
  
  let’s list all namespaces in your cluster:
  > **$ kubectl get ns**                                                                                                    
  `NAME---------|--LABELS--|-STATUS---|-AGE`                                                                             
  `----------------------------------------`                                                                                
  `default------|--none----|--Active--|-1`                                                                                
  `hkube-public-|--none----|--Active--|-1`                                                                           
  `hkube-system-|--none----|--Active--|-1h`        
  
 Creating a namespace from a custom-namespace.yaml file.
 
    apiVersion: v1
    kind: Namespace
    metadata:
     name: custom-namespace
  
  > **$ kubectl create -f custom-namespace.yaml**                                                                       
  namespace "custom-namespace" created
  
  You could have created the namespace like this:
  > **$ kubectl create namespace custom-namespace**                                                               
  namespace "custom-namespace" created
  
Deleting a pod by name
  > **$ kubectl delete pod kubia-gpu**                                                                                                                                                             
  pod "kubia-gpu" deleted
  
Deleting pod using label selectors 
> **$ kubectl delete po -l creation_method=manual**                                                                       
pod "kubia-manual" deleted                                                                                                 
pod "kubia-manual-v2" deleted                                                                             

Deleting pods by deleting the whole namespace
> **$ kubectl delete ns custom-namespacename**                                                                             
space "custom-namespace" deleted

***KEEPINGS POD HEALTHY***

Creating an HTTP-Based liveness probe

    apiVersion: v1
    kind: Pod
    metadata:
      name: kubia-liveness
    spec:
      containers:
      - image: luksa/kubia-unhealthy //this is the image the broken app
        name: kubia
        livenessProbe:               // A liveness probe that will
          httpGet:                   // perform an HTTP GET
           path: /                   // The path to rquest in the HTTP request
           port: 8080                // Yhe network port the probe should connect to
           
Create the upper's pod
> **$ kubectl create -f kubia-liveness-probe.yaml**                                                     
pod "kubia-liveness" created

To see the liveness probe
> **$ kubectl get pod kubia-liveness**                                                                                            
NAME-------------READY-----STATUS----RESTARTS---AGE                                                                                  
kubia-liveness---1/1-------Running---0----------50s                                                                                    

Configuring additional properties of the liveness probe 
> Liveness: http-get http://:8080/ delay=0s timeout=1s period=10s #success=1                                                             
                                    ➥ #failure=3

    apiVersion: v1
    kind: Pod
    metadata:
      name: kubia-liveness
    spec:
      containers:
        - image: luksa/kubia-unhealthy
        name: kubia
          livenessProbe:
            httpGet:
              path: /
              port: 8080
            initialDelaySeconds: 15 //Kubernetes will wait 15 seconds before executing the first probe.

    
  ***REPLICATION CONTROLLER***                                                                                                
  You’re  going  to  create  a  YAML  file  called  kubia-rc.yaml 
  
    apiVersion: v1
    kind: ReplicationController
    metadata:
      name: kubia
    spec:
      replicas: 3          //The desired number of pod instances
      selector:            //The pod selector determining what pods the RC is operating on
        app: kubia         
    template:              //The pod template for creating new pods
      metadata:
        labels:
          app: kubia
      spec:
        containers:
        - name: kubia
          image: luksa/kubia
          ports:
          - containerPort: 8080

To  create  the  ReplicationController,  use  the  kubectl create  command
> **kubectl create f kubia-rc.yaml**                                       
replicationcontroller "kubia" created

Seeing the ReplicationController in action
> **kubectl get pods**                                                                                                                 
NAME-------------READY-----STATUS---------------RESTARTS---AGE                                                                            kubia-53thy------0/1-------ContainerCreatting---0----------2s                                                                         
kubia-k0xz6------0/1-------ContainerCreatting---0----------2s                                                                           
kubia-g3vkg------0/1-------ContainerCreatting---0----------2s                                                                      

Geeting information about a Replication Controller       
> **kubectl get rc**                                                                                                           
NAME------DESIRED---CURRENT---READY----AGE                                                                         
kubia-----3---------3---------2--------3m                                                           

Scaling up a ReplicaitonController
> **kubectl scale rc kubia --replicas=10**                                                                                        

Deleting a replication controller with --cascade=false leaves pods unmanaged.                                                     
> **$ kubectl delete rc kubia --cascade=false**                                                                                         
replicationcontroller "kubia" deleted   

***ReplicaSets***                                                                                                         
A ReplicaSet behaves exactly like a ReplicationController, but it has more expressivepod selectors.                             

    apiVersion: apps/v1beta2 //ReplicaSets aren’t part of the v1API, but belong to the apps API group and version v1beta2.
    kind: ReplicaSet
    metadata:
      name: kubia
    spec:
      replicas: 3
    selector:
      matchLabels:   //You’re using the simpler matchLabels selector here, which is much like a ReplicationController’s selector.
        app: kubia
    template:       //The template is the same as in the ReplicationController.
      metadata:
        labels:
          app: kubia
     spec:
      containers:
        - name: kubia
          image: luksa/kubia
        
> **kubectl create -f kubia-replicaset.yaml**                                                                          
replicaset.apps "kubia" created

The main improvements  of  ReplicaSets  over  ReplicationControllers  are  their  more expressive label selectors.

    apiVersion: apps/v1beta2
    kind: ReplicaSet
    metadata:
      name: kubia
    spec:
      replicas: 3
      selector:
        matchExpressions:
          - key: app        //This selector requires the pod to contain a label with the “app” key.
            operator: In    //The label’s value must be “kubia”.
            values:
             - kubia
      template:
        metadata:
          labels:
            app: kubia
        spec:
          containers:
          - name: kubia
            image: luksa/kubia
As in the example, each expression must contain a key, an operator, and possibly (depending on the operator) a list of values. You’ll see four valid operators:
![](https://github.com/fthgul/kubernetes_commands_notes/blob/master/matchLabelReplicaSet.PNG)

ReplicaSet the same way you’d delete a ReplicationController:
> **kubectl delete rs kubia**                                                                                   
replicaset "kubia" deleted

***Services***  
kubia-svc.yaml

    apiVersion: v1
    kind: Service
    metadata:
      name: kubia
    spec:
      ports:
      - port: 80               //The port this service will be available on
        targetPort: 8080       //The container port the service will forward to
      selector:           //---All pods with the app=kubia label 
        app: kubia       // ---will be part of this service.
        
> **kubia create -f kubia-svc.yaml** 

REMOTELY EXECUTING COMMANDS IN RUNNING CONTAINERS

List  the  pods  with  the  **kubectl get pods** command and choose one as your target for the exec command (in the following example, I’ve chosen the kubia-7nog1 pod as the target). You’ll also need too btain the cluster IP of your service (using **kubectl get svc**, for example). When running the following commands yourself, be sure to replace the pod name and the service IP with your own: 

> **$ kubectl exec kubia-7nog1 -- curl -s http://10.111.249.153**                                                                       
You’ve hit kubia-gzwli

![](https://github.com/fthgul/kubernetes_commands_notes/blob/master/kubectl-exec.PNG)

EXPOSING MULTIPLE PORTS IN THE SAME SERVICE

Your service exposes only a single port, but services can also support multiple ports. For example,  if  your  pods  listened  on  two  ports - let’s  say  8080  for  HTTP  and  8443  for HTTPS—you  could  use  a  single  service  to  forward  both  port  80  and  443  to  the  pod’sports 8080 and 8443.

    apiVersion: v1
    kind: Service
    metadata:
      name: kubia
    spec:
      ports:
      - name: http
        port: 80            --//Port 80 is mapped-
        targetPort: 8080    --//to the pods’ port 8080.
      - name: https
        port: 443          --//Port 443 is mapped-
        targetPort: 8443   --// to pods’ port 8443.
      selector: 
        app: kubia         --//The label selector always applies to the whole service. 

        
     
DISCOVERING SERVICES THROUGH ENVIRONMENT VARIABLES

When a pod is started, Kubernetes initializes a set of environment variables pointing to each service that exists at that moment. If you create the service before creating the client pods, processes in those pods can get the IP address and port of the service by inspecting their environment variables.

> **$ kubectl exec kubia-3inly env**                                                                                               
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin                                                                
HOSTNAME=kubia-3inly                                                                                   
KUBERNETES_SERVICE_HOST=10.111.240.1                                                                                            
KUBERNETES_SERVICE_PORT=443                                                                       
...                                                                                                                       
KUBIA_SERVICE_HOST=10.111.249.153                                                                                                       
KUBIA_SERVICE_PORT=80                                                                                                 

**Connecting to services living outside the cluster**

 Instead  of  having  the  service  redirect  connections  to pods in the cluster, you want it to redirect to external IP(s) and port(s). Client  pods  running  in  the  cluster  can  connect  to  the  external  service  like  they connect to internal services.
 
 Introducing service endpoints
 
 Services don’t link to pods directly. Instead, a resource sits in between—the Endpoints resource.
 
 ![](https://github.com/fthgul/kubernetes_commands_notes/blob/master/img1.PNG)
 
 You may have probably realized this already, but having the service’s endpoints decoupled from the service allows them to be configured and updated manually.
 
 
 **Exposing services to external clients**
 
 ![](https://github.com/fthgul/kubernetes_commands_notes/blob/master/image2.PNG)
 
 Using a NodePort service
 
  This is similar to a regular service (their actual type is ClusterIP), but a NodePort service  can be accessed not only through the  service’s internal cluster IP, but also through any node’s IP and the reserved node port.
  
 ![](https://github.com/fthgul/kubernetes_commands_notes/blob/master/image3.PNG) 
 
 ![](https://github.com/fthgul/kubernetes_commands_notes/blob/master/image5.PNG) 
 
 > **$ kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'**                                 
35.233.247.211 35.227.187.11 34.83.123.164                                                                          

Once you know the IPs of your nodes, you can try accessing your service through them:

>**$ curl http://35.233.247.211:30123**                                                                                
You've hit kubia-vfgnh                                                                                      
 







           




      
              

   

















  
