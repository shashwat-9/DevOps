# Module 3 - Application Deployment

1. Lesson 8: Deployment
2. Lesson 9: Installing Kubernetes Application

## Lesson 8: Deployments
##### Contents
1. Running Applications as Deployment
2. Labels, Selectors, and Annotations
3. Deployment Scalability
4. Deployment Updates
5. Deployment History
6. StatefulSet
7. DaemonSet
8. Autoscaling

### 1. Running Applications as Deployment
#### Scalable Applications
 - An application in a standalone Pod cannot be scaled.
 - The Deployment is the standard resource for running stateless applications.
 - For stateful applications, StatefulSet should be used.
 - DaemonSet is a modification to a Deployment that runs one application instance on every cluster node, unless a node taint prevents it from doing so.
 
#### Deployment Features
 - Deployments use ReplicaSet to scale applications. ReplicaSet is another api-resource.
 - For automatic application scaling, use HorizontalPodAutoscaler(discussed later).
 - Deployment also offer `updateStrategy` a property that allows applications to be upgraded without experiencing any downtime.
 - Related to Deployment application updates, it offers the option to rollback to a previous version of application.
 

### 2. Labels, Selectors, and Annotations
#### Labels and Selectors
 - A label is a key-value pair that is commonly used in Kubernetes.
 1. Deployment created with `kubectl create deploy <deploymentname>` automatically get the label `app=<deploymentname>`.
 2. Pods started with `kubectl run <podname>` automatically get the label `run=<podname>`.
 - The purpose of the label is to connect different objects:
 1. Deployment tracking Pods.
 2. Services connecting Pods.
 - The `selector` is used on resources to specify which label to track.
 - As a cmd line argument, __--selector__ can be used to filter output on the presence of label.

#### Demo: Exploring Labels
 - `kubectl create deploy webapp --image=nginx --replicas=3`
 - `kubectl get deploy webapp -o yaml | less`
 - `kubectl get all --selector app=webapp`
 - `kubectl get all --show-labels`

Note: Every resource should ideally have a label.

#### Managing Labels
 - Use `kubectl label` to manually set labels.
 - Notice that when a label is set manually to a Deployment after its creation, it will not be inherited to child resources.
 - Use `kubectl label key-` to remove the label `key` with its value.
 
#### Demo: Manually managing Labels
 - `kubectl create deploy bluelabel --image=nginx`
 - `kubectl label deploy bluelabel state=demo`
 - `kubectl get deploy --show-labels`
 - `kubectl get deploy --selector state=demo`
 - `kubectl get all --selector app=bluelabel`
 
#### Demo: Manually Managing Labels
 - `kubectl describe deployment bluelabel` -> look for label
 - `kubectl describe pod bluelabel-xxx`
 - `kubectl label pod bluelabel-xxx app-` -> will remove the autoassigned run label and start a new pod to meet the requirements,
 - `kubectl get all --selector app=bluelabel`
 
#### Understanding Annotations
 - Annotations are used to store additional information in resources.
 - This can be generic information, like notes.
 - Annotations are also used by certain kubernetes commands to store automanaged information in the resource.
 
#### Demo: Exploring Annotations
 - `kubectl create deploy notes --image=nginx --dry-run=client -o yaml > notes.yaml`
 - `kubectl apply -f notes.yaml`
 - `kubectl get deploy notes -o yaml | less`
 - `kubectl annotate deploy notes newnote="my note"`
 
 - We can look for the `annotations` key in the yaml file, used to create the resource, for the annotations.
 
### 3. Deployment Scalability
#### Scaling Applications Manually
 - Deployment use ReplicaSet to manage the expected number of application instances.
 - When an application is started using `kubectl create deploy ... --replicas=3` the ReplicaSet ensures the desired number of instances is running.
 - To manually manage application scalability, use `kubectl scale <deploymentname> --replicas=3`. This will update the number of replicas.

#### Demo: Scaling Applications
 - `kubectl create deploy scaledapp --image=nginx --replicas=3`
 - `kubectl get all`
 - `kubectl delete pod scaledapp-[Tab]` -> after this, a new pod will automatically be added.
 - `kubectl get all`
 - `kubectl scale deploy scaledapp --replicas=2`
 
### 4. Deployment Updates
 - Deployments make updating applications easier.
 - To manage how applications are updated, an update strategy is used:
	- `strategy.type.rollingUpdate` updates application instances in batches to ensure application functionality continue to be offered at any time.
	- As a result of `rollingUpdate`, during the update different versions of the application will be running.
	- For applications that don't support offering multiple versions simultaneously, set `strategy.type.recreate`.
	- The `recreate` strategy will bring down all the application instances, after which the new application version is brought up. Disadvantage is a little downtime.
	
#### Managing Rolling Updates
 - To manage how `rollingUpdate` will happen, two parameters are used:
 1. `maxSurge` specifies how many application instances can be running during the update above the regular number of application instances.
 2. `maxUnavailable` defines how many applications instances can be temporarily unavailable.
 - Both parameters take an absolute number or a %age as their argument.
 
#### Demo: Managing Updates
 - `kubectl create deploy upapp --imge=nginx:1.17 --replicas=5`
 - `kubectl get deployments.app upapp -o yaml | grep -A5 strategy`
 - `kubectl set image deploy/upapp nginx=nginx:1.18; kubectl get all --selector app=upapp`
 - `kubectl edit deploy upapp` -> change strategy.type to recreate.
 - `kubectl set image deploy/upapp nginx=nginx:1.19; kubectl get all --selector app=upapp`
 
#### 5. Deploymennt History
 - Using the Deployment update procedure, the Deployment creates a new ReplicaSet that uses the new properties.
 - The old ReplicaSet is kept, but the number of Pods will be set to 0.
 - This makes it easy to roll back to previous state.
 - `kubectl rollout history` will show the rollout history of a specific deployment, which can easily be revereted as well.
 - Use `kubectl rollout history deployment mynginx --revision=1` to observe between versions.
 
#### Demo: Managing Rollout History
 - `kubectl create -f rolling.yaml` -> a rolling.yaml file is create with update strategy of rollingUpdate.
 - `kubectl rollout history deployment`
 - `kubectl edit deployment rolling-nginx` -> we can use this to update any parameters of the running deployment
 - `kubectl rollout history deployment` -> shows all the rollout history
 - `kubectl describe deployments roling-nginx` -> name of the deployment created above in the rolling.yaml file
 - `kubectl rollout history deployment rolling-nginx --revision=2`
 - `kubectl rollout history deployment rolling-nginx --revision=1`
 - `kubectl rollout undo deployment rolling-nginx --to-revision=1` -> to rollback to previous deployment
 
### 6. StatefulSet
 - StatefulSet maintains the identity of Pods, even if they are restarted.
 - This is required by Stateful applications, like databases.
 - StatefulSet is useful when one of the following is required:
	- Stable, unique network identifiers
	- Stable, persistent storage
	- Ordered, graceful deployment and scaling
	- Ordered and automated rolling Updates
 - Setting up a StatefulSet is easier when using helm charts.
 
#### StatefulSet Working
 - Storage must be provisioned by a PersistentVolume and will be claimed by the StatefulSet `volumeClaimTemplate`, which generates a PVC.
 - Deleting a StatefulSet does not delete associated volumes.
 - StatefulSets requires a headless Service(discussed later).
 - While creating the Pods, every next Pod copies the configuration of the Pod before and adds its unique identity.
 - As a result, Pods are created one-by-one, and it will take a while before all Pod instances are available.
 - It may be required to scale down the StatefulSet replicas to 0 prior to deletion.
 
#### StatefulSet Storage
 - The StatefulSet `volumeClaimTemplate` generates a PVC that connects to a PV.
 - To Connect to an existing PV, or request storage from a specific StorageClass, the `storageClassName` property is used.
	- If no `storageClassName` property is set, storage is obtained from the default StorageClass.
	- If no StorageClass is available, make sure that PVs with the appropriate `storageClassName` property are available!
 - Each Pod has its own storage; the application in the StatefulSet is expected to synchronize data between the different Pods.
 - This corresponds to common practice in databases.
 
#### Demo: Creating a StatefulSet
 - `kubectl apply -f statefulset-new.yaml`
 - `kubectl get pods,pvc,statefulset` #pod is pending as no storage is available
 - `kubectl delete -f statefulset-new.yaml`
 - edit statefulset-new.yaml and remove the storageClassName line.
 - `kubectl apply -f statefulset-new.yaml`
 - `kubectl get Pods` # it still shows a state of pending
 - `kubectl get pvc` # the PVC from the previous attempt have not been cleaned up and block new PVCs from being created.
 - `kubectl delete -f statefulset-new.yaml`
 - `kubectl delete pvc`
 - `kubectl apply -f statefulset-new.yaml` # now it works
 
 
### 7. DaemonSet
#### Understanding DaemonSet
 - A DaemonSet is a deployment that starts one pod instance on every node in the cluster.
 - This is useful in cases where a software component like an agent needs to be available on the cluster nodes.
 - When nodes are added or removed, the DaemonSet automatically changes the number of Pods accordingly.
 - Use YAML code to create DaemonSet. There's no imperative way of doing this.
 
### 8. Autoscaling
 - For CKAD, you need to know how to manually scale Pods using `kubectl scale` etc.
 - In real cluster, Pods are often automatically scaled based on resource usage properties that are collected by the Metric Server.
 - The HorizontalPodAutoscaler observes usage statistics, and after passing a threshold, will add additional replicas.
 - setting up autoScaling is NOT required of CKAD, but as it is important in real cluster you"ll find a demo on how to do it.
 - Metric Server is required for this, which can easily be setup in minikube.
 - Setting up Metric Server in other cluster types is covered in CKA.
 
 - `minikube enable metric-server` enables metric server.
 - `kubectl get hpa`
 - There's something called load-generator perhaps.
 
 
## Installing Kubernetes Applications
1. Running Applications from yaml
2. The Helm Package Manager
3. Managing Applications with helm
4. Using Kustomize

### 1. Running Application from yaml
#### Running Applications from yaml files
 - A Kubernetes application often is a collection of resources.
 - To manage applications in a consistent way, all related resources can be defined in one yaml file.
 - While doing so, it is recommended to use clear separators between the application components.
 - Although this works, the solution is not very portable.
 - To make application management more flexible, site specific values should be separated from generic configurarions.

#### The Helm Package Manager
 - Helm is a Kubernetes package manager and is used to streamline installing and managing Kubernetes applications.
 - Helm consists of the helm tool, which needs to be installed, and a chart.
 - A chart is a Helm package, which contains the following:
 1. A description of the package
 2. One or more template containing Kubernetes manifest files.
 - Charts can be stored locally, or accessed from remote Helm repositories.
 
#### Demo: Installing the helm Binary
 - Fetch the binary from `https://github.com/helm/helm/releases`; check for the latest release.
 - `tar xvf helm-xxxx.tar.gz`
 - `sudo mv linux-amd64/helm /usr/local/bin`
 - `helm version`
 - Helm supports auto completion, using `source <(helm completion bash)` on the bash.
 
#### Helm Charts
 - Helm doesn't come with a default repository.
 - The main site for finding Helm charts, is through `https://artifacthub.io`
 - Search for specific software here, and run the commands to install it; for instance, to run the Kubernetes Dashboard:
	1. `helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/`
	2. `helm install kubernetes-dashboard kubernetes-dashboard/kubenetes-dashboard`
 - The last command creates the local application name "kubernetes-dashboard" by installing the application "kubernetes-dashboard" from the repo "kubernetes-dashboard".
 
#### Demo: Using Helm repositories
 - helm repo add bitnami https://charts.bitnami.com/bitnami
 - helm repo list # lists all the repos and details
 - helm search repo bitnami #searches for the word "bitnami"
 - helm search repo file #searches for the word "file"
 - helm search repo nginx --version #shows different versions
 
#### Installing Helm charts
 - After adding repositories, use `helm repo update` to ensure access to the most up-to-date information.
 - Use `helm install <name> <chart>` to install the chart with default parameters.
	- Notice that a chart may be installed multiple times, which is why it's important to assign the right name.
 - After installation, use `helm list` to list currently installed charts,
 - Optionally, use `helm delete` to remove currently installed charts.
 
#### Demo: Installing Helm charts
 - `helm install bitnami/mysql --generate-name`
 - `kubectl get all` -> shows the current installed into upping state
 - `helm show chart bitnami/mysql`
 - `helm show all bitnami/mysql`
 - `helm list [--all-namespaces]`
 - `helm status mysql-xxxx`
 
 
### 3. Managing Application with helm
 - A Helm chart consists of templates to which specific values are applied.
 - The values are stored in the values.yaml file, within the Helm chart.
 - Use `helm show values` to list current values(a lot!).
 - Read documentation on artifacthub.io for a list of all values.
 - Create a custom values.yaml which will be merged with the generic values.yaml : `helm install ... --values values.yaml`
 - When upgrading an application, you'll have to use `--value values.yaml` as well.
 
#### Overriding Values
 - The default values.yaml file which is a part of the helm chart contains default values which are defined as a key-value pair.
 - To get information about useful values, check the documentation of the Helm chart: artifacthub.io is a good place to do so.
 - While instaling the chart, you can use a custom values.yaml file, provided with `--values values.yaml` as an argument to `helm install` to overwrite the default values.
 - Alternatively, use `helm install ... -set key=value` to set individual values.
 - Best practice: use a values.yaml file and only use --set when absolutely necessary.
 
#### Demo: Providing Custom Parameters
 - `helm repo add bitnami https://charts.bitnami.com/bitnami`
 - `helm show values bitnami/nginx`
 - `helm show values bitnami/nginx | grep commonLabels`
 - `helm show values bitnami/nginx | grep replicaCount`
 - `vim values.yaml`
	- `commonLabels: "type: helmapp"`
	- `replicaCount: 3`
 - `helm install bitnami/nginx --generate-name --values values.yaml`
 - `helm list`
 - `helm get values nginx-xxxx` #show custom values only
 - `helm get values --all nginx-xxxx` #show all values
 
#### Helm Upgrades
 - A Helm chart is the package you install from the Helm repository.
 - A Helm installation is an installed instance of that package.
 - A Helm release is a combination between a chart and an installation.
 - You can upgrade either of these.
 - For instance, to first run a nginx installation without exposing it, use `helm install bitginx bitnami/nginc --set ingress.enabled=false`
 - Next, to make the application accessible, use `helm update bitginx bitnami/nginx --set ingress.enabled=true`
 - Use `helm repo update` to fetch the latest version of charts from the repository.
 - Use `helm upgrade bitginx bitnami/nginx` to upgrade to the latest version of the application.
 
#### 4. Using Kustomize
 - kustomize is a kubernetes feature that uses a file with the name kustomization.yaml to apply changes to a set of resources.
 - This is convenient for applying changes to input files that the user does not control himself, and which contents may change because of new versions appearing in Git.
 - Use `kubectl apply -k ./` in the directory with the kustomization.yaml and the files it refers to apply changes.
 - Use `kubectl delete -k ./` in the same directory to delete all that was created by the kustomization.
 
#### Understanding a Sample kustomization.yaml
```
	resources:	#defines which resources(in YAML files) apply
	- deployment.yaml
	- service.yaml
	namePrefix: test- 	#specifies a prefix should be added to all names
	namespace: testing #objects will be created in this specific namespace
	commonLabels:	#labels that will be added to all objects
		environment: testing 
```

#### Using Kustomization Overlays -1
 - Kustomization can be used to define a base configurarion, as well as multiple deployment scenarios(overlays) as in dev, staging, and prod, for instance.
 - In such a configuration, the main __kustomization.yaml__ defines the structure:
```
	- base
		- deployment.yaml
		- services.yaml
		- kustomization.yaml
	- overlays
		- dev
		 - kustomization.yaml
		- staging
		 - kustomization.yaml
		- prod
		 - kustomization.yaml
```
 - In each of the overlays/{dev,staging,prod}/kustomization.yaml, users would reference the base configurarion in the resources field, and specify changes for that specific environment:
 ```
	resources:
	- ../../base
	namePrefix: dev-
	namespace: development
	commonLabels:
		environment: development
 ```
 
#### Demo: Using Kustomization
 - `cat deployment.yaml`
 - `cat services.yaml`
 - `kubectl apply -f deployment.yaml services.yaml`
 - `cat kustomization.yaml`
 - `kubectl apply -k .`