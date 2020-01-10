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

      
              

   

















  
