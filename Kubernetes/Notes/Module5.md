# Module 5 - Application Environment, Configurations and Security

1. ConfigMaps and Secrets
2. Deploying applications the DevOps way
3. Working with the API
4. Security

## 1. ConfigMaps and Secrets
 Learn how to decouple site specific information using generic application code using configMaps and Secrets
 
 1. Why Decoupling is important
 2. Providing Variables to kubernetes applications
 3. Providing Variables with ConfigMaps
 4. Providing Configuration Files using ConfigMaps
 5. Secrets
 6. Configuring Application to use Secrets
 7. Secrets and Authenticated Registry access.
 
### 1.1 Why Decoupling is important
To provide configurations to applications in a flexible way, we can use configMaps. So, we've containerized application image, which could run on various differnet environments.
 - We can have the Configurations for each environment in a yaml file, and that yaml file will be stored in configMaps.
 
### 1.2 Providing Variables to kubernetes applications
 - Providing variables while starting applications is not useful in fully automated environments.
	- Configuration as Code startegies require the variables to be included in configuration files.
 - Kubernetes does not offer a command Line option to provide variables while running a Deployment with `kubectl create deploy`
	- First, use `kubectl create deploy mydb --image=mariadb`
	- Next, use `kubectl set env deploy mydb MYSQL_ROOT_PASSWORD=password`
 - While running a Pod, environment variables can be provided, but you shouldn't run standalone pods,
   - `kubectl run mydb --image=mysql --env="MYSQL_ROOT_PASSWORD=password`

   
#### Demo
 - `kubectl create deploy mydb --image=mariadb`
 - `kubectl describe pods mydb-xxx-yyy`
 - `kubectl logs mydb-xxx-yyy`
 - `kubectl set env deploy mydb MARIADB_ROOT_PASSWORD=password`
 - `kubectl get deploy mydb -o yaml>mydb.yaml` # Clean it, as a running deployment yaml have the current status of the application as well.
 
### 1.3 Providing Variables with ConfigMaps
 - The configMap is an API resource to store site specific information.
 - It has two differnet uses:
	1. Variable Storage
	2. Configuration file(s) storage upto a size of 1 MiB.
 - If bigger amounts of data are needed, they should be stored in a Pod Volume.

#### Using Variables in ConfigMaps
 - Use `kubectl create cm` to create a ConfigMap,
	- `--from-literal key=value` (to provide variable on cmd line)
	- `--from-env-file=/path/to/file` (to provide variables via a confifuration file)
 - An environment file is a file that has multiple variables defined on different lines.
 - Add `--dry-run=client -o yaml` to generate YAML code instead of creating the resource.
 - The easy way to use variables from ConfigMaps is using `kubectl set env`.
 - `kubectl set env --from=config/<mycm> deploy/<mydeploy>
 - While using `kubectl set env`, the `--prefix` option can be used to put a prefix before the variable as defined in the configMaps. It's like, in the config file we have, `root_password` and the prefix is assigned as `mariadb`, so the final config would be `mariadb.root_password`
 
#### Understanding ConfigMap variable use
 - `pod.spec.containers.env.name`: defines the variable name.
 - `pod.spec.containers.env.valueFrom.configMapKeyRef.name` refers to the name of the COnfigMap.
 - `pod.spec.containers.env.valueFrom.configMapKeyRef.key` defines the key in the ConfigMap from which the value must be set to the variables.
 
#### Demo: Working with ConfigMaps
 - `kubectl create deploy mydb --image=mariadb --replicas=3`
 - `kubectl create cm mydbvars --from-literal=ROOT_PASSWORD=password`
 - `kubectl get cm mydbvars -o yaml`
 - `kubectl set env deploy/mydb --from configmap/mydbvars --prefix=MARIADB_`
 - `kubectl get deploy mydb -o yaml | grep env -A 5`
 
### 1.4 Providing Configuration Files using ConfigMaps
 - ConfigMap can contain multiple confifuration file.
 - In the data section of the configMap, each file is referred to with its own key.
 - To use configuration files from ConfigMaps, the configMap needs to be used as a Pod volume and mounted on a directory.
 
#### Demo: Using ConfigMaps for the configuration Files
 - `echo "hello world" > index.html`
 - `kubectl create cm myindex --from-file=index.html`
 - `kubectl describe cm myindex`
 - `kubectl create deploy myweb --image=nginx`
 - `kubectl edit deploy myweb`
 ```
	spec.template.spec
	volumes:
	-name: cmvol
	configMap:
	 name: myindex
	spec.template.spec.containers
	volumeMounts:
	-mountPath: /usr/share/nginx/html`
	 name: cmvol
 ```

### 1.5 Secrets
 - A Secret is a base-64 encoded alternative for a ConfigMap.
 - Secret types are used for typical scenarios:
 1. `generic`: used for generic sensitive values like passwords
 2. `tls`: stores TLS keys
 3. `docker-registry`: used to store registry access credentials.
 - Using secrets makes kubernetes more secure, as the actual value itself doesn't have to be stored in the application manifest file.
 - Using secrets doesn't make the data completely secure, as the values are encoded and not encrypted.
 - Kubernetes internally works with Secrets to deal with sensitive values.
 
### 1.6 Configuring applications to use secrets

#### Using Secrets in Applications
 - Secrets and ConfigMaps are more or less same, just in secrets we need to provide the type as well.
 - There're different use cases for using Secrets in applications:
	- To provide TLS keys to the application: `kubectl create secret tls my-tls-keys --cert=tls/my.crt --key=tls/my.key`
	- To provide security to passwords: `kubectl create secret generic my-secret-pw --from-literal=password=verysecret`
	- To provide access to SSH private key: `kubectl create secret generic my-ssh-key --from-file=ssh-private-key=.ssh/id_rsa`
	- To provide access to sensitive files, which would be mounted in the application with root access only: `kubectl create secret generic my-secret-file --from-file=/my/secretfile`
 - As a secret basically is an encoded ConfigMap, it is used in a similar way to using ConfigMaps in applications.
 - If it contains variables, use `kubectl set env`
 - If it contains files, mount the secret.
 - While mounting the secret in the Pod spec, consider using `defaultMode` to set permissionMode: `...volumes.secret.defaultMode: 0400`
 - Notice that mounted secrets are automatically updated in the application when the secret is updated.
 
#### Demo: Using a secret to provide passwords
 - `kubectl create secret generic dbpw --from-literal=ROOT_PASSWORD=password`
 - `kubectl describe secret dbpw` # secret 
 - `kubectl get secret dbpw -o yaml` # It will give us the secret value in base64 encoded form
 - we can use `echo $encoded_pwd | base64` to get the raw password
 - `kubectl create deploy mynewdb --image=mariadb`
 - `kubectl set env deploy mynewdb --from=secret/dbpw --prefix=MYSQL_`
 
 - Secret lies in the `etcd`, and the ordinary users don't have access to it. So for reading the content in a Secret manually will require a read permission on Secrets in the etcd, which is typically the kubeadmin user.
 `spec.containers.env` shows env variables and where they're fetched from.

### 7. Secrets and Authenticated Registry Access

#### Using Secret for Registry Access
 - Many container registries require authenticated access.
 - To provide the access credentials, they can be stored in the `docker-registry` Secret type.
 - While creating a `docker-registry` type Secret, two options are available:
	- Specify all required values on the command line.
	- Import the values from the config.json file that has been generated by logging in from the command line.
 - To use `docker-registry` type secrets, the `pod.spec.imagePullSecrets` key is used.
 
#### Demo: Using Secrets for Registry Access
 - `kubectl create secret docker-registry dockercreds --docker-username=unclebob --docker-password=secretpw --docker-email=uncle@bob.org --docker-server=https://index.docker.io/v1/`
 - `kubectl get secrets dockercreds -o yaml`
 - `kubectl run gittools --image=docker.io/sandervanvugt/gittools --dry-run=client -o yaml > gittools.yaml`
 - In `pods.spec.imagePullSecrets` add the list item `name: dockercreds` to __gittools.yaml__. 
 - `kubectl create -f gittools.yaml`
 
## 2. Deploying Applications the DevOps way

1. DevOps and GitOps
2. Blue/Green Deployments
3. Canary Deployments

### 1. DevOps and GitOps
 - DevOps is a lot. While working with Kubernetes, DevOps focuses on a few items:
 1. Configuration as Code (YAML files)
 2. Continuous access to applications
 3. A methodology that can easily be reproduced
 4. Zero-downtime updates
 - GitOps brings a higher level of automation to DevOps.
 1. YAML files are provided by a Git repository
 2. A GitOps operator picks up changes and applies them to the Kubernetes cluster in an automated way.
 
### 2. Blue/Green Deployment
 - Blue/Green Deployments are a way of working to ensure that applications can be safely be upgraded to a new version.
 - The essence of Blue/Green Deployment is that the new version of the application can already be tested while the old version is still being used.
 - There are many different ways in which Blue/Green Deployment can be implemented:
 1. Using kubernetes Services
 2. Using kubernetes Ingress
 3. Using advanced resource such as Istio
 
#### Service versus Ingress Blue/Green
 - Service-based Blue/Green is happening at a lower level.
 - Existing client connections may get disturbed.
 - For better support of existing connections, Ingress can be used.
 - Notice that Ingress only works for HTTP/HTTPS-based application.
 
#### Demo: Ingress Based Blue/Green Deployments
Part1: Creating the Blue Application
 - `cd ~/ckad/kustomize-bluegreen/blue`
 - `cat *`
 - `kubectl apply -k .`
 - `kubectl get deploy,pods,svc,cm,img`
Part2: Testing application access
 - `sudo sh -c "echo $(minikube ip) myapp.local >> /etc/hosts"`
 - In browser: `http://myapp.local`
Part3: Creating the Green Application
 - `cd ~/ckad/kustomize-bluegreen/green`
 - `cat *`
 - `kubectl apply -k .`
Part4: Making the switch
Here, we just changed the service direction towards green service, in the ingress. The green folder don't have an ingress service, and hence the change followed.
 - `sed -i -e's/blue-svc/green-svc/' myapp-ing.yaml
 - `kubectl apply -f myapp-ing.yaml`
 - `curl http://myapp.local`
 - `kubectl scale deploy blue-deploy --replicas=0`
 
#### Demo: Service based Blue/Green Deployments
 - `kubectl create deploy blue-nginx --image=nginx:1.14 --replicas=3`
 - `kubectl expose deploy blue-nginx --port=80 --name=bgnginx`
 - `kubectl get deploy blue-nginx -o yaml > green-nginx.yaml`
	- clean up dynamic generated stuff(metadata last lines, and status)
	- Change `image` version
	- Change "blue" to "green" throughout
 - `kubectl create -f green-nginx.yaml`
 - `kubectl get pods`
 - `kubectl delete svc bgnginx; kubectl expose deploy green-nginx --port=80 --name=bgnginx`
 - `kubectl delete deploy blue-nginx`
 
### 3. Canary Deployment

#### Understanding Canary Deployments
  - A canary Deployment upgrade strategy will expose a new version of the application to a limited number of users before completing the migration to a new version.
  - This allows user exposure with minimized risk.
  - If things don't work out well, it's easy to revert to the previous situation by just removing the new application instance(s).
  
#### Services versus Ingress Canary
 - Canary Deployments can be configured based on Services or Ingress.
 - Using Ingress is preferred as the application picks up the change without reconnecting.
 - Service-based canary deployments are configured to use a common `selector` lable on the old as well as the new applications.
 - Ingress-based Canary Deployments are using two Ingress resources pointing to the same Ingress virtual host.
 - Canary Deployment solutions are also offered by alternative ecosystem solutions.
 
###### How's Ingress is used Blue/Green Deployment?
 - So in ingress, we have annotations where we can add `canary: true` and `canaryweight: 10` means new deployment will handle 10% load.
 - The Ingress controller shall distribute the traffic accordingly. We can have old and new ingress rules, where the distribution in the mentioned %age will be done. (read about, it's unclear right now, how one ingress controller uses two ingress rules).
 
#### Demo : Ingress Canary Deployment
 - `echo new-version > index.html`
 - `kubectl create cm new-version --from-file=index.html`
 - `echo old-version > index.html`
 - `kubectl create cm old-version --from-file=index.html`
 - `kubectl apply -f canary.yaml`
 - `kubectl expose deploy old-nginx --port=80 --type=NodePort`
 - `sed -i -e 's/old/new/' canary.yaml` # everywhere change old to new in the canary.yaml file
 - `kubectl apply -f canary.yaml`
 - `kubectl expose deploy new-nginx --port=80 --type=NodePort`
 - `kubectl get deploy,pods,svc`
 
 - `sudo sh -c "$(minikube ip) theapp.info >> /etc/hosts"
 - `kubectl create ing old-version --rule="theapp.info/=old-version:80"`
 - `curl theapp.info` #all traffic goes to old-version
 - `cat new-ing.yaml` # notice the annotations
 - `kubectl apply -f new-ing.yaml`
 - `curl theapp.info` # repeat at least 15 times
 
 
#### Service based canary Deployment
 - Services have a selector which looks for labels in all the deployments.
 - So, let's say we have new and old deployments in our cluster, we can increase the replicas of the old and reduce the replicas of new. That's how canary deployment can be achieved as Service-based.
 
#### Demo: Service-based Canary Deployment
 - `echo "old-nginx" > index.html`
 - `kubectl create cm old --from-file=index.html`
 - `echo "new nginx" > index.html`
 - `kubectl create cm new --from-file=index.html`
 - `cat canary.html`
 - `kubectl apply -f canary.yaml`
 - `sed -i -e 's/old/new/' canary.yaml`
 - `sed -i -e 's/replicas: 3/replicas: 1/' canary.yaml`
 - `kubectl apply -f canary.yaml`
 - `kubectl expose deploy old-nginx --name=theapp --port=80 --selector type=canary --type=NodePort`
 - `kubectl get svc`
 - `curl $(minikube ip):<nodePort>` # repeat at least 10 times
 
## 3. Working with the API

1. Understanding the Kubernetes API
2. Using curl to work with API Objects
3. Understanding API Deprecations
4. Extending the API
5. CustomResourceDefinition
6. Operator

### 1. Understanding the kubernetes API
 - The Kubernetes API provides a way to interact with Kubernetes.
 - It provides RESTful endpoints that allow the users to perform operations on the cluster.
 - The API uses resources to represent components of the Kubernetes cluster.
 - It supports a declarative configuration model, where users define the desired state of the cluster in YAML or JSON manifests.
 - It allows for authentication and authorization, using different solutions.
 - The API is extensible, which allows for addition of new resources to the Kuberenetes environment.
 - The main API documentation is here: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.30 (version could be updated)
 - Each API group can have its own version number. See here for more information: https://kubernetes.io/docs/reference/using-api/api-overview/
 - Use `kubectl api-resource` for information about resource type.
 - Or `kubectl api-version` for resource and version information. What version of which resource is supported on the current installation.
 
#### Communicating with the API
 - The `kube-apiserver` provides access to the API.
 - The `kubectl` client is the main tool to communicate with the API.
	- It uses the `~/.kube/config` file to use the TLS keys for secure communication.
 - The `kube-proxy` can be used as a proxy that uses the .kube/config TLS keys for secure communication.
 - This allows utilities like `curl` to communicate with the API in a non-secured way.

#### Client Configuration
 - Use `kubectl config view` to view current `kubectl` client configuration.
 - This command reads the `~/.kube/config` file, which contains the following elements:
	- `Cluster`: certificate and API endpoint needed to contact the kube-apiserver process.
	- `Users`: the TLS certificate that make up the user account.
	- `Context`: the Combination of user and cluster, to which a default namespace is added. If we set the context to another namespace say xyz, then the default namespace would be xyz, instead of default.

### 2. Using curl to work with API Objects
 - To access the API using curl, start the kube-proxy on the kubernetes user workstation.
 ```kubectl proxy --port=8001&
  curl https://localhost:8001
 ```
 - This shows all the available API paths and groups, providing access to all exposed functions.
 
#### Demo: Using curl to Access API resources
 - On the host that runs kubectl: `kubectl proxy --port=8001 &`
 - `kubectl run proxypod --image=nginx`
 - `curl http://localhost:8001/version`
 - `curl http://localhost:8001/api/v1/namespaces/default/pods` -> shows the pods
 - `curl http://localhost:8001/api/v1/namespaces/default/pods/proxypod` -> show direct API access to a Pod.
 - `curl -XDELETE http://localhost:8001/api/v1/namespaces/default/pods/proxypod` will delete the httpd pod(proxypod).
 
### 3. Understanding API Deprecations
 - With new Kubernetes release, old API versions may get deprecated.
 - If an Old version gets deprecated, it will be supported for a minimum of two or more Kubernetes releases.
 - When you see a deprecation message, make sure to take action and change your YAML manifest files!

#### Demo: Dealing with Deprecations 
```
	kubectl create -f redis-deploy.yaml
	kubectl api-versions
	kubectl explain --recursive deploy (recursive shows child objects and its objects)
```

### 4. Extending the API
 - The kubernetes API can be extended in different ways,
	- Using the CustomResourceDefinition API resource
	- Using Custom Controller
	- Using API Aggregation
	
#### CustomResourceDefinition
 - A CustomResourceDefinition is an API resource that makes adding your own API resources easy.
 - The crds are integrated with the kubernetes API server and follow the API server update mechanism.
 - Using crd is common, as it provides an easy way to add resources without any need to program them.

#### Custom Controller
 - A controller is a process that watches for changes in the Kubernetes API.
 - When changes occur, the controller takes action to ensure that the desired state of the resource is maintained.
 - Controller allow you to automate common tasks within the Kubernetes cluster.
 - Custom controllers communicate with the kubernetes API by using client libraries.
 
#### API Aggregation
 - In API aggregation, the Kuberenetes API is extended with additional API servers.
 - API aggregation enables a higher level of customization, but API servers needs to be programmed.
 - The aggregated API server can integrate resources from multiple sources.
 
### 5. CustomResourceDefinition

#### Understanding CustomResourceDefinitions
 - CustomResourceDefinitions allow users to add custom resources to cluster.
 - Doing so allows anything to be integrated in a cloud-native environment.
 - The crd allows users to add resources in a very easy way.
   - The resources are added as extension to the orginal kubernetes API server.
   - No Programming skills required.
 - Adding custom resources only makes sense if you have an application that's using them!
 
#### Creating Custom Resources
 - Creating Custom Resource using crds is a two-step procedure.
  - First, you'll need to define the resource, using the CustomResourceDefinition API kind.
  - After defining the resource, it can be added through its own API resource.
  
#### Demo: Creating Custom Resources
 - `cat crd-object.yaml`
 - `kubectl create -f crd-object.yaml`
 - `kubectl api-resources | grep backup`
 - `cat crd-backup.yaml`
 - `kubectl create -f crd-backup.yaml`
 - `kubectl get backups`
 
 - The name of the CRDs should always be FQN.

### 6. Operator

#### Understanding Operator and Controllers
 - Operators are custom applications, based on CustomResourceDefinitions.
 - Operators can be seen as a way of packaging, running, and managing applications in Kubernetes.
 - Operatores are based on Controller, which are Kubernetes components that continuously operate dynamic systems.
 - The Controller loop is the essence of any controller.
 - The Kubernetes Controller manager runs on a reconciliation loop, which Continuously observes the current state, compares it to the desired state, and adjusts it when necessary.
 - Operators are application specific controllers.
 - Operators can be added to Kubernetes by developing them yourself.
 - Operators are also available from community websites.(Operatorhub.io)
 - A common registry from the Kubernetes ecosystem are provided as operators:
	- Prometheus: a monitoring and alerting solution
	- Tigera: the operator that manages the calico network plugin.
	- Jaeger: used for tracing transactions between distributed services.
	
## 4. Security

1. Authentication and Authorization
2. API Access and ServiceAccounts
3. Role Based Access Control (RBAC)
4. SecurityContext
5. Resource Requests, Limits, and Quotas

### 1. Authentication and Authorization

#### Understanding Authentication
 - Authentication is about where Kubernetes users come from.
 - In vanilla Kubernetes and Minikube, a local Kubernetes admin account is used for authentication.
 - In more advanced setups, you can create your own user accounts(Covered in CKA).
 - The kubectl config specifies to which cluster to authenticate.
	- Use `kubectl config view` to see current settings.
 - The config is read from `~/.kube/config`.
 
#### Understanding Authorization
 - Authorization is what these users do.
 - Behind authorization, there is Role Based Access Control (RBAC) to take care of the different options.
 - Use `kubectl auth can-i ...`(like `kubectl auth can-i get pods`) to find out what you can do.
 - Use `kubectl auth can-i get pods --as=system:serviceaccount:bellevue:viewer -n bellavue`
 
#### Demo: Showing Current Authorizations
 - `kubectl auth can-i get pods`
 - `kubectl auth can-i get pods --as bob@example.com`
 
### 2. API Access and ServiceAccounts
 - All actions in a Kubernetes Cluster need to be authenticated and authorized.
 - ServiceAccounts are used for basic authentication from within the Kubernetes Cluster.
 - RBAC is used to connect a ServiceAccount to a specific role.
 - Every Pod uses the default ServiceAccount to contact the API server.
 - This default ServiceAccount allows a resource to get information from the API Server, but not much else.
 - Each Service account uses a Secret to automate API Credentials.
 
#### Custom ServiceAccount Use Case
 - Most pods do fine with the default ServiceAccount.
 - If a Pod needs access to resources in the cluster, a custom ServiceAccount that uses a RoleBinding to connect to a specific Role is needed.
 - For instance, this is needed for network plugins, monitoring software and other additional components installed in Kubernetes.
 
#### Demo: Exploring Service Accounts
 - `kubectl describe pod anypod` #look for the Service Account
 - `kubectl get sa -n default`
 - `kubectl describe pod coredns -n kube-system` #look for ServiceAccount
 - `kubectl get sa -n kube-system`
 
### 3. Role Based Access Control(RBAC)

#### Understanding RBAC
 - RBAC uses 3 components to grant permissions to API objects.
	- A Role consists of Verbs which assign specific permissions like view, edit, and more.
	- A ServiceAccount is used by Pods that need access to API resources.
	- The RoleBinding connects a ServiceAccount to a Role.
 - Roles and RoleBinding have a Namespace scope, ClusterRoles and ClusterRoleBindings have a cluster Scope.
 - In RBAC, users can be used for people that needs access to specific resources(not in CKAD).
 
#### Demo: Configuring RBAC
 - `kubectl create ns bellevue`
 - `kubectl create role viewer --verb=get --verb=list --verb=watch --resource=pods -n bellevue`
 - `kubectl create sa viewer -n bellevue`
 - `kubectl create rolebinding viewer --serviceaccout=bellevue:viewer --role=viewer -n bellevue`
 - `kubectl create deploy viewginx --image=nginx --replicas=3 -n bellevue`
 - `kubectl set serviceaccount deployment viewginx viewer -n bellevue`
 - `kubectl auth can-i get pods --as=system:serviceaccount:bellevue:viewer -n bellevue`
 
#### Demo: Exploring RBAC Usage
 - `kubectl describe serviceaccount coredns -n kube-system`
 - `kubectl describe clusterrolebinding system:coredns`
 - `kubectl describe clusterrole system:coredns`
 
### 4. SecurityContext

#### Understanding SecurityContext
 - A SecurityContext defines privilege and access control settings for a Pod or container, and includes the following:
	- `allowPrivilegeEscalation`: whether or not a container can run with root privileges.
	- `capabilities`: POSIX capabilities used by the container
	- `runAsNotRoot`: enforces a non-privileged UID.
	- `readOnlyFileSystem`: no writes to the container filesystems.
	- `runAsUser`: runs as a specific User.
	- `seLinuxOptions`: specifies SELinux Context labels.
	
 - Use `kubectl explain pod.spec.[containers.]SecurityContext for a complete overview.
 
#### Using SecurityContext
 - Notice that SecurityContext can be applied to Pods as well as containers.
 - When SecurityContext prevents a Pod from running successfully, use `kubectl describe` to get additional information from the events.
 - Expect Pods that fail because of SecurityContext restrictions to show a status of Pending or failed.
 
#### Demo: Using SecurityContext
 - `kubectl apply -f securitycontextdemo.yaml`
 - `kubectl exec -it security-context-demo -- sh` The name is defined in the above file. sh opens up the shell, as it has the privileges.
 - `kubectl get pods` shows failing pods
 - `kubectl describe pods nginxsecure`
	- note that the image wants to run as root, which is not allowed, which is why the pod will never run.
	
 - We can change the `runAsNotRoot` to true/false to see the restrictions posed by privileges. Then we can use `kubectl describe pod <podname>`, and look at the events.
 
### 5. Resource Requests, Limits, and Quotas
 - Resource requests can be set for containers in a pod to ensure that the pod is only scheduled on cluster nodes that meet the resource requests.
	- Use `pod.spec.containers.resources.requests` to set.
 - Resource limits can be set for pods to maximize the use of system resources.
	- Use `pod.spec.containers.resources.limits` to define.
 - Quota are restrictions that can be set on a Namespace to maximize the availabilty of resources within that Namespace.
 
 
	