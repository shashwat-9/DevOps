## Module 2 - Kubernetes Essentials

1. Pod Basic Features
2. Pod Advanced Features
3. Kubernetes Storage

### Lesson 5 : Pod Basic Features
1. Options for Running Kubernetes Applications
2. Pod Structure and Use
3. Running Pods the DevOps way
4. Generating YAML files
5. Multi-container Pods
6. Namespaces
7. Pod TroubleShooting

#### 1. Options for Running Kubernetes Applications
 - The pod is the minimal entity to run Kubernetes applications. We run containers inside a pod.
 - To run stateless applications, using Deployments is recommended. It adds :
 1. Adds scalability
 2. Adds easy zero-downtime application updates
 - For stateful applications, the StatefulSet is used.
 1. Additional requirements make using StatefulSet a bit more complicated.
 - The __helm__ package manager makes it easy to run applications and all required dependencies.
 
#### 2. Pod Structure and Use
 - The pod is the minimal entity managed by Kubernetes. Kubernetes doesn't manages any containers, but pods.
 - Pods add Kubernetes properties to containers. These're properties that allows you to orchestrate containers in a cluster, or a cloud environment, like 
 1. nodeselector which allows selection of the node that runs the Pod.
 2. priority allows for adding priority labels.
 3. restartPolicy tells the cluster what to do if the Pod fails.
 - From a Pod, one or more containers can be started.
 - Pods can also include volumes, which make it easy to provide storage to containerized applications.
 - We can use `kubectl explain pod.spec`.
 
 Standalone pods may look nice, but there's a couple of disadvantages.
##### Standalone Pods Disadvantages
 - Standalone pods are not rescheduled in case of failure. So, if anything bad happens to it, it won't be restarted.
 - Rolling updates don't apply to standalone Pods; you can only bring the pods down and bring it up again with the new settings.
 - Standalone Pods cannot be scaled.
 - They also cannot be replaced automatically.

##### Running Pods
 - Standalone Pods are commonly used for testing and troubleshooting
 - Even in the exam, the questions are from standalone pods.
 - To deploy permanent applications, Deployments and StatefulSets are recommended.
 - To start a standalone Pod, use `kubectl run podname --image=imageName`
 - Use `kubectl get pods` for an overview if currently running pods.
 - `kubectl describe pod podname` shows properties of currently running Pods.
 
##### 3. Running Pods the DevOps Way
 - In DevOps, the purpose is to deploy applications in a consistent way that is easy to reproduce.
 - To do so, configuration is often provided as code.
 - This works for Kubernetes resources: instead of running Kubernetes commands, YAML manifest files are used, from which applications can be started.
 - Creating resources based on YAML files is referred to as the declarative way of working with Kubernetes.
 - The imperative way of working is where resources are created from the command line, using `kubectl run` or `kubectl create`.
Tip : In real-life, the declarative way of working is used a lot. On the exam, it maybe faster to use the imperative way.


###### What is YAML
 - YAML is a human-readable data-serialization language.
 - It is perfect for DevOps to create input files to manage objects in different systems like Kubernetes and Ansible.
 - It uses indentation to identify relations.
 - When combined with git, its an excellent way to manage and update configurations in a structured way.
 
###### Kubernetes YAML Ingredients
 - All of the YAML manifest ingredients are defined as resource properties in the API.
 a. apiVersion: specifies which version of the API to use for this object.
 b. kind: indicates the type of object(Deployment, Pod, etc)
 c. metaData: contains administrative information about the object.
 d. spec: contains the specifics to define exactly how the resources is used.
 e. status: added by the cluster for running resources.
 - Use `kubectl explain` to get more information about the basic properties.
 
###### pod.spec.containers components
 - In the container spec, different parts are commonly used:
 1. name: the name of the container.
 2. image: the image that should be used
 3. command: the command the container should run.
 4. args: arguments that are used by the command.
 5. env: env variables that should be used by the container.
 - These are all a part of the pod.spec.containers, which can be checked with `kubectl explain`.
 
###### Creating Resources from YAML files
 - `kubectl create -f resources.yaml` creates resources from a YAML file. If the resource already exists, it fails.
 - `kubectl apply -f resource.yaml` creates the resources as defined in the YAML file. If the resources already exists, it updates properties that need updating.
 - `kubectl delete -f resource.yaml` deletes the resources as defined in the YAML file.
 - `kubectl replace -f resource.yaml` is less commonly used and replaces a current resource with the resource specification as in the YAML file.
 
##### 4. Generating YAML files
 - DO NOT write YAML files, generate them.
 - To generate YAML files, add `--dry-run=client -o yaml > my.yaml` as an argument to the `kubectl run` and `kubectl create` commands
 - If we don't use the redirect(`>`) operator at the end, it will write everything on the console. 
 - `kubectl run myginx --image=nginx --dry-run=client -o yaml > mynginx.yaml`
 - Next, create the resources in the YAML using `kubectl apply -f ...`

##### 5. Multi-Container Pods
 - The one-container Pod is standard.
 - Single container Pods are easier to build and maintain. By single container, it means only one type of container, like only webserver or database, not both. This way updating one, doesn't include the downtime for other.
 - Specific settings can be used that are good for a specific workload.
 - To create applications that consists of multiple containers, microservices should be defined.
 - In a microservice, different independently managed pods are connected by resources that can be provided by kubernetes.
 - There're some defined cases where you might want to run multiple containers in a Pod:
 a. Sidecar container: a container that enhances the primary application, for instance for logging.
 b. Ambassador container: a container that represents the primary container to the outside world, such as proxy.
 c. Adapter container: a container that is used to adopt the traffic or data pattern to match the traffic or data pattern in other applications in the cluster.
 
###### Sidecar containers
 - A sidecar container is providing additional functionality to the main container, where it makes no sense running this functionality in a separate Pod.
 - Think of logging, monitoring, and syncing.
 - The essence is that the main container and the sidecar container have access to the shared resources to exchange information.
 - Istio service mesh is a common addition to Kubernetes that injects sidecar container in pods for traffic management. Istio is all about advanced traffic management.
 - Often shared volumes are used for this purpose (discussed in-depth later).
 
##### 6. Namespaces
 - Kubernetes Namespaces resources leverage Linux kernal namespace to provide resource isolation.
 - Different Namespaces can be used to strictly separate between customer resources and thus enable multi-tenancy.
 - Namespaces are used to apply different security-related settings,
 a. Role-Based Access Control(RBAC)
 b. Quota
 - By installing complex Kubernetes applications in their own Namespace, managing them is easier.
 
###### Managing Namespaces
 - To show resources in all Namespaces, use `kubectl get ... -A`
 - To run resources in a specific Namespace, use `kubectl run ... -n namespace`
 - Use `kubectl create ns nsname` to create a Namespace.
 - `kubectl run podname --image=imageName -n nspace`, this cmd runs the pod in the specified namespace.
 - Similarly, `kubectl get pods -n nspace`, will look for pods in the nspace namespace and the default namespace.
 - To delete the namespace, we can use `kubectl delete -n nspace pod podname`
 
##### 5. Pod TroubleShooting
 - Use `kubectl get pods` to check on current Pod status.
 - Use `kubectl describe pod ...` to get more information about Pod Status in the Kubernetes cluster for a failing Pod.
 - Use `kubectl logs podname` to get access to pod application output that has been logged.
 - Use `kubectl exec -it podname sh` to open a shell on the pod and analyze specific components.
 
