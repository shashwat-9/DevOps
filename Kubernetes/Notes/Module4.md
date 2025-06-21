# Module 4 - Services and Networking

1. Networking
2. Ingress and Gateway API

## 1. Networking
 - Provide access to the applications that are running in your cluster.

1. Kubernetes Networking
2. Services
3. Creating Services (using kubectl and manifest file)
4. Service Resources in Microservices
5. Services and DNS
6. NetworkPolicy
7. Advanced Networking: Gateway API and Istio

### 1. Kubernetes Networking
 - Let's suppose, we've `n` nodes in our cluster. All of these nodes are connected to an external network, for exposing to the internet.
 - Further, Kubernetes uses something called as software defined network, which are internal networks.
 - There're 2 such internal networks, 
 1. Cluster network : used for cluster administrative tasks 
 2. Pod Network : The network to which all the pods are connected, and each have an ip assigned.
 - But pod network is an internal network, and can't be used by an external user to access the application.
 - Therefore, comes the services, which sits before the pod network, making it accessible to the external user. They act as a load balancer for the pods.
 - There're 2 types of services,
 1. Node Port : Exposes the Service on each node’s IP at a specific port (30000–32767 range), which if undefined manually, will be allocated dynamically. Accessible externally via <NodeIP>:<NodePort>, which is taken care by the external DNS LB. This doesn't interact directly to pods.
 2. Cluster IP : Exposes the Service within the cluster via a virtual IP. Use case: Internal communication between apps (e.g., frontend to backend). It's a default service, only accessible within the cluster (not externally unless combined with other mechanisms).
 
### 2. Services

#### Services
 - A Service is an API resource that is used to expose a set of pods. Used for networking.
 - Services are applying round-robin load balancing to forward traffic to specific pods.
 - The set of pods that is targeted by a service is determined by a selector(which is a label).
 - The `kube-controller-manager` will continuously scan for Pods that match the `selector` and include these in the service.
 - If pods are added or removed, they immediately show up in the services. Like within 2 - 3 seconds.

#### Services and Decoupling 
 - Services exist independently from the applications they provide access to.
 - The Service needs to be created independently of the application, and after removing an application, it also needs to be removed separately.
 - The only thing they do is watch for Pods that have a specific label set matching the `selector` that is specified in the service.
 - That means that one service can provide access to Pods in multiple Deployments(deploying the same labelled pods), and while doing so, Kubernetes will automatically load balance between these pods.
 - This strategy is used in canary deployments(covered later).
 
#### Service Types
 - `ClusterIP`: this default type exposes the service on an internal cluster Ip address.
 - `NodePort`: allocates a specific port on the node that forwards to the service IP address on the cluster network.
 - `LoadBalancer`: provisions an external load balancer to handle incoming traffic to applications in public cloud.(AWS ELBs)
 - `ExternalName`: works on DNS names; redirection is happening at a DNS level, which is useful in migration.
 - `Headless`: a Service used in cases where direct communication with Pods is required, which is used in StatefulSet.
 
 For CKAD, focus on `ClusterIP` and `NodePort`.
 
### 3. Creating Services
 - `kubectl expose` can be used to create Services, providing access to Deployments, ReplicaSets, Pods, or other services.
 - In most cases, `kubectl expose` exposes a Deployment, which allocates its pods as the service endpoint.
 - `kubectl create service` can be used as an alternative solution to create services.
 - While creating a service, the `--port` argument must be specified to indicate the port on which the service will be listening for incoming traffic.
 
#### Service Ports
 - While working with services, different ports are specified:
 - `targetPort`: the port on the application(container) that the service addresses.
 - `port` : the port on which the service is accessible.
 - `nodePort` : the port that is exposed externally while using the `NodePort` service type.
 
#### Demo: Creating Services
 - `kubectl create deployment nginxsvc --image=nginx`
 - `kubectl scale deployment nginxsvc --replicas=3`
 - `kubectl expose deployment nginxsvc --port=80`
 - `kubectl describe svc nginxsvc`
 - `kubectl get svc nginxsvc -o yaml`
 - `kubectl get svc`
 - `kubectl get endpoints`
 - `minikube ssh`
 - `curl http://svc-ip-address` (replace the ip with that of service) -> this works cuz clusterIP is accessible within the minikube.
 - `exit`
 - `kubectl edit svc nginxsvc`
	```...
	protocol: TCP
	nodePort: 32000
	type: NodePort
	```
 - `kubectl get svc`
 - (from host): `curl http://$(minikube ip):32000
 
### 4. Service Resources in Microservice
 - In a microservice architecture, different frontend and backend Pods are used to provide the application.
 - Backend Pods(like datatbases) should be exposed internally only, using the `ClusterIp` Service type.
 - Frontend Pods(like Webservers) should be exposed for external access, using the `NodePort` Service type or the ingress resource.
 - For more advanced traffic management in microservices, a service mesh can be used.
 
### 5. Services and DNS
 - Exposed Services automatically register with Kubernetes internal coredns DNS Server.
 - The standard DNS name is composed as `servicename.namespace.svc.clustername`
 - As a result, Pods within the same namespace can access servicename by using its short name.
 - To access servicenames in other Namespaces, the fully qualified domain name must be used.
 
#### Demo: Services and DNS
 - `kubectl describe svc -n kube-system kube-dns`
 - `kubectl create ns elsewhere`
 - `kubectl run nginxpod --image=nginx -n elsewhere`
 - `kubectl expose -n elsewhere pod nginxpod --port=80`
 - `kubectl run testpod --image=busybox --sleep infinity`
 - `kubectl exec -it testpod -- cat /etc/resolv.conf
 - `kubectl exec -it testpod --wget --spider --timeout=1 nginxpod`
 - `kubectl exec -it testpod --wget --spider --timeout=1 nginxpod.elsewhere.svc.cluster.local`
 
### 6. NetworkPolicy
 - By default, there are no restrictions to network traffic in K8s.
 - Pods can always communicate, even if they're in other namespaces.
 - To limit this, NetworkPolicies can be used.
 - NetworkPolicies need to be supported by the network plugin though,
	- The Weave plugin does NOT support network policies!
	- Calico is a common plugin that does support NetworkPolicy.
 - If in a policy there is no match, traffic will be denied.
 - If no NetworkPolicy is used, all traffic is allowed.
 
#### NetworkPolicy Identifiers
 - In NetworkPolicy, three different identifiers can be used:
	- `podSelector`: specifies a label to match Pods.
	- `namespaceSelector`: used to grant access to specific namespaces.
	- `ipBlock`: marks a range of IP addresses that is allowed. Notice that the traffic to and from the node where a pod is running is always allowed.
 - When defining a Pod- or Namespace-based NetworkPolicy, a selector label is used to specify what traffic is allowed to and from the Pods that match the `selector`.
 - NetworkPolicies do not conflict, they're additive.
 
#### Demo: Using NetworkPolicy
 - `kubectl get pods -n kube-system | grep -i calico`
 - `kubectl apply -f nwpolicy-complete-example.yaml`
 - `kubectl expose pod nginx --port=80`
 - `kubectl exec -it busybox -- wget --spider --timeout=1 nginx` -> will fail, as access is not true
 - `kubectl label pod busybox access=true`
 - `kubectl exec -it busybox -- wget --spider --timeout=1 nginx` -> will work
 
### 7. Advanced Networking: Gateway API and Istio

#### What is Gateway API
 - Gateway API provides routing and traffic management policies.
 - It is an advanced layer that uses custom resources for managing incoming(ingress) and outgoing(egress) traffic.
 - Gateway API adds features that are not addressed by Ingress, including:
	- support for TCP, UDP and gRPC
	- Traffic splitting and mirroring
	- Websockets protocols
 - Future developments seem to further integrate Gateway API functionality.
 
#### What is Istio?
 - Istio is a service mesh and makes managing complex relations between applications in a microservice easier.
 - As such, it provide rules for managing traffic in microservices.
 - It includes features for traffic management, security, and observability in the service mesh.
 - Its focus is on service-to-service communication.
 - Istio maybe used in addition to core Kubernetes networking and is Optional. Make sense to use this in large scale environment.

#### Assignment
 - Create a namespace with the name "remote"
 - In the remote namespace, run an Nginx POd with name "remoteweb" and expose it such that it can be reached on the minikube host port 31999.
 - In the default namespace, run the pod testpod, based on the BusyBox image and using the __sleep____infinity__ command as its default command.
 - From the testPod pod, use `wget --spider --timeout=1` to verify you can access the default webpage of the remoteweb Pod.
 - Also verify that this web page is accessible on minikube host port 31999.
 

## 2. Ingress and Gateway API

1. Managing Incoming Traffic
2. Ingress Components
3. Installing Ecosystem Ingress Controllers
4. Using Minikube Ingress Controller
5. Using Ingress
6. Configuring Ingress rules
7. Understanding Gateway API
8. Configuring Gateway API
9. Using Gateway API to Provide Access to Applications.
10. Troubleshooting Networking

### 1. Managing Incoming Traffic
 - For a long time, Ingress has been the solution to manage incoming traffic.
 - Recently, Ingress has gone into "feature freeze" and will be replaced by Gateway API.
 - Currently, Ingress is still in the exam objective, this is expected to be replaced with Gateway API in the future.
 - For that reason, in this lesson you'll learn about both.
 
 Warning: Do not configure Ingress and Gateway API on the same machine! They are most likely going to bite one another.
 
### 2. Ingress Components
 
#### Understanding Ingress
 - Ingress is used to provide external access to internal Kubernetes cluster resources.
 - To do so, Ingress uses an external load balancer.
 - This load balancer is implemented by the ingress controller which is running as a Kubernetes application.
 - As an API resource, Ingress uses services to connect to Pods that're used as a service endpoint.
 - To access resources in the cluster, the host name resolution (DNS or /etc/hosts) must be configured to resolve to the Ingress load balancer IP.
 - Ingress exposes HTTP and HTTPS routes from outside the cluster to Pods within the cluster.
 - Traffic routing is controlled by rules defined on the Ingress resource. And these rules function as configuration file. So, the ingress controller is running as a pod, and where normally your ingress controller would read a configuration file, if on a standalone computer. On cloud, an ingress resource is used to provide the configuration.
 - Ingress can be configured to do the following(all of them or a subset):
 1. Give services externally-reachable URLs
 2. Load balance traffic
 3. Terminate SSL/TLS
 4. Offer name based virtual hosting
 
#### Understanding Ingress Controller
 - Creating Ingress resources without an Ingress controller has no effect.
 - Many Ingress controller exist:
 1. `nginx`: http://kubernetes.github.io/ingress-nginx/
 2. `haproxy`: http://www.haproxy.com/blog/dissecting-the-haproxy-kubernetes-ingress-controller/
 3. `traefik`
 4. `kong`
 5. `contour`
 
 The Ingress Controller should be accessible from the outside. These controllers run on the nodes(one or more) as pods.
 The Ingress controller gets the configuration from a configuration file, and the kubernetes alternative for this resource is Ingress resource. In this ingress resource, rules are defined. Based on these rules, Ingress controller match certain services.
 - Ingress is only for HTTP(S).
 
#### Installing Ecosystem Ingress Controller
 - From the kubernetes ecosystem, different Ingress controller are provided.
 - As the CNCF doesn't want to favor specific ecosystem projects, vanilla Kubernetes doesn't come with an Ingress controller.
 - Kubernetes distributions normally provide one or more supported Ingress controller.
 - Alternatively, Ingress controller may be installed manually.
 - In vanilla Kubernetes, you have to install an Ingress controller or else you can't use it.
 
#### Demo: Installing an Ingress Controller
 - On vanilla Kubernetes only!
  - `helm upgrade --install ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx --create-namespace
  - `kubectl get pods -n ingress-nginx`
  
### 4. Using the Minikube Ingress Controller

#### Minikube Ingress
 - Minikube is a kubernetes distribution and comes with addons to integrate third-party solutions.
 - Use `minikube addons list` to show available addons.
 - Use `minikube addons enable` to enable a specific addon.
 
#### Demo: Using the minikube Ingress Addon
 - `minikube addons list`
 - `minikube addons enable ingress`
 - `kubectl get ns`
 - `kubectl get all -n ingres-nginx`
 
### 5. Using Ingress

#### Demo: Configuring Ingress Rules
 - `kubectl create deploy nginxsvc --image=nginx --port=80`
 - `kubectl expose deploy nginxsvc`
 - `kubectl create ingress nginxsvc-ingress --rule="/=nginxsvc:80"`
 - `echo "$(minikube ip) nginxsvc.info" >> /etc/hosts` -> on linux
 - `kubectl describe ing nginxsvc-ingress`
 - `curl nginxsvc.info`
 
 Note: we can always use `-h` with ingress, like `kubectl create ing -h`
 - In the command to create Ingress `kubectl create ingress simple --rule="foo.com/bar=svc1:8080"`, the name of the ingress component is simple, and the rules says, foo.com will forward the request to service svc1.
 
### 6. Configuring Ingress Rules
 - Each Ingress Rule contains the following:
 a. An Optional host to be used as a name-based virtualhost. If no host is specified, the rule applies to all inbound HTTP traffic.
 b. A list of paths (like /testPath). Each path has its own backend. Paths can be exposed as a regex.
 c. The backend, which is a Service resource. It is common to configure a default backend in an Ingress controller for incoming traffic that doesn't match a specific path.
 
#### Ingress Paths
 - The ingress `pathType` specifies how to deal with path requests.
 - The __Exact__ `pathType` indicates that an exact match should occur,
	- If the path is set to `/foo`, and the request is `/foo/` there is no match.
 - The __Prefix__ `pathType` indicates that the requested path should start with,
	- If the path is set to `/`, any requested path will match.
	- If the path is set to `/foo`, `/foo` as well as `/foo/` will match.
	
#### Ingress Types
 - Ingress backed by a single Service: there's one rule that defines access to one backend service.
 `kubectl create ingress single --rule="/files=fileservice:80"`
 - Simple fanout: there are two or more rules defining different paths that refer to different services,
 `kubectl create ingress single --rule="/files=fileservice:80" --rule="/db=dbservice:80"
 - Name-based virtual hosting: there are two or more rules that route requests based on the host header.
	- Make sure there is a DNS entry for each host header.
	- ```kubectl create ingress multihost --
		rule="my.example.com/files*=fileservice:80" --
		rule="my.example.org/data*=dataservice:80"
		```
		
#### Demo: Name-based Virtual Hosting
 - `kubectl create deploy mars --image=nginx`
 - `kubectl create deploy saturn --image=httpd`
 - `kubectl expose deploy mars --port=80`
 - `kubectl expose deploy saturn --port=80`
 - Add entries to /etc/hosts  -> for linux
  - $(minikube ip) mars.example.com
  - $(minikube ip) saturn.example.com
  - `kubectl create ingress multihost --rule="mars.example.com/=mars:80" --rule="saturn.example.com/=saturn:80"`
  - `kubectl edit ingress multihost`, change `pathType` to `Prefix`
  - `curl saturn.example.com; curl mars.example.com`
  
 
### 7. Understanding Gateway API
 - In current Kubernetes, Ingress is in feature freeze and no longer developed.
 - The replacement is Gateway API.
 - Gateway API adds more advanced features to manage incoming traffic:
	- Advanced traffic management
	- More options that are integrated in the API resources
 - Gateway API may find its way into future versions of the CKAD exam.
 
#### Gateway API Resources
 - Gateway API uses specific API resources which are provided as CRDs(CustomResourceDefinition). It's a way to add our own objects to kubernetes.
	- GatewayClass: represents the Gateway Controller.
	- Gateway: defines an instance of traffic handling infrastructure.
	- HTTPRoute: defines how traffic is routed to one or more services.
 - To work with Gateway API, a Gateway API controller needs to be installed.
 - Without this controller, there's nothing actually handling the incoming traffic.
 
#### Gateway API Controller
 - Different Gateway API controller are provided by the ecosystem.
 - In this class, we'll use the Nginx Gateway Fabric, which is easily installed with __helm__.
 - Before installing the controller, you must install the custom resources.
 
### 8. Configuring Gateway API

#### Resources: GatewayClass
 - The GatewayClass resource represents the physical Gateway Controller.
 - It uses `spec.controllerName` to connect to a specific Gateway Controller.
 - It has no further configuration, the real configuration is done on the Gateway resource.
 
#### Gateway
 - Multiple Gateways can connect to one Gateway Controller.
 - At least one Gateway is required.
 - The Gateway uses the `gatewayClassName` property to connect to the GatewayController.
 - It also defines `listener` to specify which protocols should be serviced.
 
#### HTTPRoutes
 - This defines to which service an incoming request should be forwarded.
 - Incoming requests are identified by the `spec.hostnames`.
 - The `parentRefs` property connects the HTTPRoute to a Gateway.
 - The `backendRefs` property connects the HTTPRoute to a Service.
 
User -> DNS -> (Gateway API, Controller, Service) -> Gateway Class -> Gateway -> HTTPRoute -> K8s Service
 - All the custom resources in the above flow is to be installed.
 
### 9. Using Gateway API to provide Access to Applications

#### Understanding the Procedure
 - First, you'll need to make sure the required custom resources are available.
 - Next, install a community Gateway API controller.
 - Verify the community Gateway Controller is ready to accept incoming requests.
 - Create Kubernetes application you want to provide access to.
 - Configure GatewayClass, Gateway, and HTTPRoute.
 - Test by accessing the Service that exposes the community controller.
 
#### Demo: Using Gateway API
 - Install CRD's: `kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/release/download/v1.1.0/standard-install.yaml` -> if anything is going wrong, check the latest version
 - Install nginx-gateway-fabric controller: `helm install ngf oci://ghcr.io/nginxinc/charts/nginx-gateway-fabric --create-namespace -n nginx-gateway`
 - Verify: `kubectl get pods,svc -n nginx-gateway`
 - At this point, you have a GatewayController. Use `kubectl get gc` and notice the name(nginx).
 - Make the gwc service accessible as a NodePort: `kubectl edit -n nginx-gateway svc ngf-nginx-gateway-fabric` and set the type to NodePort.(It was set as loadbalancer first, but we reset it to nodePort).
 
#### Demo: Using Gateway API
 - Create Kubernetes Resources
  - `kubectl create deploy nginxgw --image=nginx --replicas=3`
  - `kubectl expose deploy nginxgw --port=80`
 - Open http-routing.yaml from course Git repository and verify
	- `gatewayClassname: nginx`
	- `backendRefs.name: nginxgw`
 - `kubectl apply -f http-routing.yaml`
 - Let's do a first test, using port-forwarding
  - `sudo sh -c "echo 127.0.0.1 whatever.com >> /etc/hosts/"
  - `kubectl -n nginx-gateway port-forward ngf-nginx-gateway-[Tab] 8080:80 8443:443`
  (keep on running, we may choose `ctrl + z` and then bg to keep it running in the background)
  - `curl whatever.com:8080`
 - At this point, we've used a small trick to bypass the Service and access the gwf Pod directly.
 - Now change the IP address for `whatever.com` in `/etc/hosts` to match the Minikube IP address (use minikube Ip).
 - Use `kubectl get svc -n nginx-gateway` to find the `nodePort` the Service uses.
 - Next, use `curl whatever.com:32000` to address the Service on this NodePort.
 - As in real life, there would be an external load balancer that load balances traffic between all different Ports on the nodes.
 
 
  