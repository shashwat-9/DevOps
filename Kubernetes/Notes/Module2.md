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
 - To delete a pod in a namespace, we can use `kubectl delete -n nspace pod podname`
 
##### 5. Pod TroubleShooting
 - Use `kubectl get pods` to check on current Pod status.
 - Use `kubectl describe pod ...` to get more information about Pod Status in the Kubernetes cluster for a failing Pod.
 - Use `kubectl logs podname` to get access to pod application output that has been logged.
 - Use `kubectl exec -it podname sh` to open a shell on the pod and analyze specific components.
 - We can use the `kubectl describe pod podname`, and in the events sections, look for events that're carried out for initiating the pod, and this can help in finding the errors/issues.


### Lesson 6 : Pod Advanced Features
1. Init containers
2. SideCar containers
3. Using Port Forwarding to Access Pods
4. restartPolicy
5. Jobs
6. CronJobs
7. Cleaning up Resources

#### 1. Init Containers
 - An init container is a special case of multi-container Pod, where the init container runs to completion before the main container is started.
 - Starting the main container depends on the success of the init container, if the init container fails the main container will never start.
 - In the yaml file, `initContainers` key is used, to specify the related images, cmds, variables and etc.
 
#### 2. Sidecar Containers
 - A sidecar container is an `initContainer` that has the `restartPolicy` field set to `Always`.
 - It doesn't occur as a specific attribute, to create a sidecar you need to create an `initContainer` with the `restartPolicy` set to `Always`.
 - The sidecar container will be started before the main Pod is started and is typically used to repeatedly run a command.
 - Unlike init containers, sidecar containers start with the main container and are expected to run alongside it throughout the Pod’s life.
 
#### 3. Using Port Forwarding to Access Pods

##### Port Forwarding
 - Pods can be accessed in multiple ways.
 - A very simple way is by using port forwarding to expose a Pod port on the kubectl host that forwards to the Pod.
 - The cmd `kubectl port-forwarding fwnginx 8080:80` will expose the port 8080 on the kubectl host and forward that to port 80 on the pod.
 - Port forwarding is useful for testing pod accessibilty on a specific cluster node, not to expose it to external users.
 - Regular user access to applications is provided through services and Ingress(Covered Later).
 
##### Demo: Port Forwarding
 - `kubectl get pods -o wide` -> lists Ip addesses as well.
 - `kubectl run fwginx --image=nginx`
 - `kubectl port-forward fwginx 8080:80 &` -> & is used for running the process in the background.
 - `curl localhost:8080`
 - We need port forwarding, cuz the pod have their own IP addresses, and thus when we do curl on the localhost, a certain port must forward it to required port in pod.

#### 4. RestartPolicy
 - The Pod `restartPolicy` determines what happens if a container that is managed by a Pod crashes.
 - If set to the default value `restartPolicy=always`, the container will be restarted after a crash or normal exit, both.
 - `restartPolicy=always` does not affect the state of the entire Pod.
 - If the Pod is stopped or Killed, `restartPolicy=always` won't start.
 
##### Demo: restartPolicy
 - `kubectl run nginx1 --image=nginx`
 - `kubectl get pods nginx1 -o yaml | grep restartP`
 - `kubectl delete pods nginx1`
 - `kubectl get pods`
 - `kubectl run nginx2 --image=nginx`
 - `minikube ssh` -> enters into the shell terminal for the kubernetes
 - `crictl ps | grep nginx2` -> crictl stands for container runtime interface
 - `crictl stop $(crictl ps | awk '/nginx2/{print $1}')`
 - `exit`
 - `kubectl get pods`
 
#### 5. Jobs
 - A Job starts a Pod with the `restartPolicy` set to never/onFailure. If we omit this, Kubernetes set it to Never.
 - To create a Pod that runs to completion, use Jobs instead.
 - Jobs are useful for one-shot tasks, like backup, calculation, batch processing and more.
 - Use `spec.ttlSecondsAfterFinished` to clean up completed Jobs automatically. It won't be listed in the `kubectl get jobs`, if the timeout has passed.
 
##### Job Types
 - 3 different job types can be started, which is specified by the completions and parallelism parameters:
 1. Non-parallel Jobs: one Pod is started, unless the Pod fails
 ```
	completion=1
	parallelism=1
 ```
 2. Parallel Jobs with a fixed completion count: the Job is complete after successfully running as many times as specified in `jobs.spec.completion`
 ```
	completion=n
	parallelism=m
 ```
 Like m pods were started, out of which n completes, then the job will be successful.
 3. Parallel Jobs with a work queue: multiple Jobs are started, when one completes successfully, the Job is complete.
 ```
	completions=1
	parallelism=n
 ```
 - The concept is simple, parallelism defines the number of pods that can run concurrently, and completion marks the number of pods that should complete their execution, irrespective of parallelism.
 - To create job, we can use `kubectl create job jobname --image=imagename -- date`
 - `kubectl get jobs,pods`, lists jobs and pods both
 - After the resouce application is completed, we can use the `kubectl delete job jobname`. And likewise, for pod.
 - To do advanced configs in kubernetes, we can generate the yaml files for the required cmd, and tweak the required params there in the yaml.
 
#### 6. CronJobs
 - Jobs are used to run a task a specific number of times.
 - A cronJob adds a schedule to a Job.
 - To add the schedule, Linux crontab syntax is used.
 - When running a cronjob, a job will be scheduled.
 - This Job, on its turn, will start a Pod.
 - To test a CronJob, use `kubectl create job myjob --from=cronjob/mycronjob`

##### Demo: Running CronJobs
 - `kubectl create cronjob -h | less`
 - `kubectl create cronjob runme --image=busybox --schedule="*/2****" -- echo greetings from your cluster`
 - `kubectl create job runme --from=cronjob/runme`
 - `kubectl get cronjobs,jobs,pods`
 - `kubectl logs runme-xxx-yyy`
 - `kubectl delete cronjob runme`
 
 If we don't clean the cronJob, new jobs will keep on getting created. So, if we're testing, and are not in production, we may delete the cronjob.
 

##### Cleaning up resources
 - Resources are not automatically cleaned up.
 - Some resources have options for automatic cleanup if they're no longer used.
 - Periodic manual cleanup maybe required.
 - If a Pod is managed by deployments, the deployment must be removed, not the Pod.
 - Try not to force resource to deletion, it may bring them in an unmanageable state. Never ever do it.
 
##### Demo: Cleaning up resources
 - `kubectl delete all`
 - `kubectl delete all --all`
 - `kubectl delete all --all --force --grace-period=-1` -> dangerous cmd
 - Don't do this: `kubectl delete all --all -A --force --grace-period=-1` -> This is stupid, as it will delete all the resources in all the namespaces.
 
 
### Lesson 7 : Kubernetes Storage
1. Ephemeral and Persistent Storage
2. Configuring Pod Volume Storage
3. Configuring PersistentVolumes
4. StorageClass
5. Configuring PersistentVolumesClaims
6. Configuring Pod Storage with PV and PVC

#### 1. Ephemeral and Persistent storage

##### Understanding Ephemeral Storage
 - When a container is started, the container working environment is created as a directory on the host that runs the container.
 - In this directory, a subdirectory is created to store changes inside the container.
 - This subdirectory is ephemeral and disappears when the container disappears.
 - The ephemeral storage is host-bound, and that doesn't work well in a cloud environment where multiple application instances are running.
 
##### Cloud Storage Needs
 - To provide persistent storage, the data needs to be stored separately.
 - Also, cloud storage should not be host-bound.
 - When cloud storage is host-bound, it needs to be synchronized when replicated pods run on different nodes.
 - Pod Volume are a pod property that allow containers to connect to any storage type that is defined within Pod.
 - PersistentVolumes are independent API resources and can be discovered dynamically while running Pods.

#### 2. Configuring Pod Volume Storage
 - Pod Volumes are defined as properties of Pods.
 - Many types of storage can be addressed using volumes: see pod.spec.volumes for a list.
 - Using Pod volumes works if Pods are used in an environment where a specific type of storage is used.
 - For more flexibility, PersistentVolumes can be used.
 - To use a Pod volume, the container needs to mount it, using `pod.spec.container.volumeMounts`
 - There is no easy command to create a Pod with Volumes, use the documentation to set it up.

##### Common Pod Volume Types
 - `emptyDir` creates a temporary directory on the host that runs a Pod and is ephemeral.
 - `hostPath` refers to a persistent directory on the host that runs the Pod.
 - `PersistentVolumeClaim` connects to available PersistentVolumes(covered later).
 - Other volume types `fc` and `iscsi` may make more sense in real life, but requires additional setup(and for that reason are not on CKAD).
 
##### Demo: Creating a Pod with a Volume
 - We can directly copy the required yaml from the doc, or can use the doc directly as well, like `kubectl apply -f https://k8s.io/example/pods/storage/redis.yaml`
 - Use `kubectl describe pods redis` and check its configuration, which contains `emptyDir` storage, mounted on `/data/redis`
 - 