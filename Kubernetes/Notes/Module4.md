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
 - `kubectl exec -it testpod -- wget --spider --timeout=1 nginxpod`
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
 - As such, it provides rules for managing traffic in microservices.
 - It includes features for traffic management, security, and observability in the service mesh.
 - Its focus is on service-to-service communication.
 - Istio maybe used in addition to core Kubernetes networking and is Optional. Make sense to use this in large scale environment.


 