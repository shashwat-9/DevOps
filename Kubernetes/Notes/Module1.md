# Module 1 - Container Fundamentals

1. Lesson 1 : Understanding and Using Container
2. Lesson 2 : Managing Container Images
3. Lesson 3 : Understanding Kubernetes
4. Lesson 4 : Creating a Lab Environment

#### Lesson 1 : Understanding and Using Container

##### What's a Container?
 - Containers provide a method to run applications in full isolations.
 - To provide this isolation, Linux Kernal namespaces are used.
 - To provide all dependencies for running the application, a container is started from an image.
 - Container images are typically obtained from a registry.
 - Container images are highly standardized and can be used on stand-alone computers as well as in cloud.
 
##### Container Runtimes
 - A container runtime is the program that starts a container.
 - __runc__ is a common container runtime.
 - To manage a container that runs on a stand-alone computer, a container engine is used on top of the container runtime.
 - Docker and Podman are the most common container engines for running stand-alone containers.
 - To run containers in cloud, Kubernetes is used as the container engine.
 - As Kubernetes is doing much more than just taking care of the running container, it is called container orchestrator.
 __runc__ is the vital part of the container running, and once it runs the image, whether it's on a computer or cloud, is managed by Docker/Podman and Kubernetes respectively.
 
##### Using Registries
 - A registry is used as a container image-store.
 - Local registries can be used to provide private access to images.
 - public registries can be used to provide access to container images for a worldwide audiences.
 - The Docker registry is the most common registry and is often used as the default registry.
 - To run container images from specific registries, a Fully Qualified Container (image) Name (FQCN) can be used at all times.
 - Registry access maybe free, some registries require login.
 
##### Starting Containers
 - To start a container, you need to refer to the image that is needed.
 - The exact way to start a container depends on the Container engine used:
	a. docker run nginx (will fetch the image from the default registry)
	b. podman run docker.io/library/nginx (requires a FQCN)
	c. kubectl run <podname> --image=docker.io/library/nginx (again, requires FQCN)
 - Monitoring Kubernetes containers will be covered later.
 
#### Docker vs Podman
 - Docker started in 2013 and rapidly became the de facto standard for running containers using __docker__ CLI.
 - The podman project was integrated in Red Hat Enterprise Linux 8 in 2018 and has since rapidly gained popularity.
 - The podman CLI provides all functionality that is provided by the docker CLI.
 - If specific docker functionality is not found in podman, this is considered a bug by the podman team.
 - While using kubernetes, neither podman nor Docker is needed, as kubernetes is its own container manager.
 
#### Managing Containers
 - The docker CLI provides many commands for managing containers.
 - Demonstrated a bunch of docker cmds.
 
#### Container Logging
 - Container don't always connect to a STDOUT.
 - As a result, if the container primary application generates output, you won't see it.
 - Container application output and errors are logged to the container logs.
 - Use `docker logs <container_name>` to see the logs of a container.
 
### Managing Container Images
1. Container Image Architecture
2. Managing Container Images
3. Image Creation Options
4. Using Dockerfile to build custom Images
5. Creating Images from Running Containers

##### 1 Container Image Architecture
 - While creating application image, common system images are often used as the foundation.
 - This can be seen in dockerfile, where the `FROM` statement is used to refer to the system image that is to be used.
 - The modifications made by adding the specific applications are stored in separate layers.
 - This makes working with container images efficient.

##### 2 Managing Container Images
 - Before starting a container, required images are pulled and stored.
 - Images can be pre-fetched using the `docker pull` command.
 - Use `docker images` for a list of currently stored images.
 - According to the image pull policy, a new image may automatically be pulled if it's available
	a. The default image pull policy in Docker is set to always.
	b. In kubernetes, it can be set to newer.
 - To remove unused images, use `docker image prune`.
 
##### 3 Image Creation Options
 - Containers have become the standard for distributing applications.
 - Creating custom images are easy.
 - `docker commit` allows you to create a custom image by saving changes made to a running container.
 - DockerFile A.K.A ContainerFile builds a custom image based on components defined in the file.
 - buildah is an advanced solution to build custom images in a scripted way.
 
##### 4. Base image
 - To create custom images, base images are normally used.
 - A base image is a minimized image that can be used as the foundation for building your own image.
 - Often, the base image is fully functional, minimized Linux Distribution.
 - However, it should not be expected that all tools are always available.
	a. Common tools like `ps` are often missing as they are not needed to run the main container application.
	b. Common base images are: a. BusyBox b. Alpine c. Red Hat UBI(Universal Base Image)

##### 5. Using Dockerfile to Build Custom Images
 - Dockerfile can be provided by application Developers.
 - In podman environment, Dockerfile is referred to as Containerfile, there are no functional differences.
 - It's also relative easy to write your own.
 - To build an image from Dockerfile, use `docker build -t <imageName>`
 - In this command, -t(tag) specifies the name of the image you want to create.
 - `.` refers to the current directory as the directory where the Dockerfile is found.
 
##### 6. Creating Images from Running Containers
 - With this way, we can create an image with the current changes in the running container.
 - We can make the changes inside the container(like creating a directory etc)
 - Then we can close it, and use the cmd `docker commit <image_name>:<desired_tag_name>`
 
### Understanding Kubernetes
1. Cloud Native Computing
2. How Kubernetes enables cloud native Computing
3. Kubernetes Origin
4. Kubernetes and Cloud Native Computing Foundation
5. Kubernetes Architecture
6. Essential API resources

#### 1. Cloud Native Computing
 - To provide access to applications on the Internet, the applications should be hosted in cloud.
 - While being hosted in the cloud, the application is decoupled from specific servers.
 - If an application is disconnected from any specific server, facilities must be provided to take care of specific features.
	a. Access to configurations
	b. Persistent Storage
	c. Application access
 - Cloud Native computing is taking care of all of these.
 
#### 2. How Kubernetes enables cloud native computing
 - Kubernetes is the open-source platform that allows containers to be used in a cloud native environment.
 - To do so, it orchestrates containers, which means that it ensures that containers are running where they need to be running.
 - Kubernetes also provides scalability: ensuring a sufficient amount of containers are running to deal with the current workload.
 - The Kubernetes API defines a set of resource type, such as pods, Deployment, and ConfigMaps that allow for storing information in a cloud native environment where no relation to specific server exists.

#### 3. Kubernetes Origin
 - Google Borg is at the origin of kubernetes.
 - It is a platform that Google had been using since the early 2000's to offer Google applications in a cloud-based environment.
 - Based on Google Borg, Google open-sourced a platform-agnostic solution for orchestrating containers.
 - Kubernetes was first announced in June of 2014, and the source code was donated to Cloud Native Computing Foundations in 2015.
 
#### 4. Kubernetes and Cloud Native Computing
 - The Cloud Native Computing Foundation (CNCF) is a part of the Linux Foundation.
 - Its core mission is to provide open-source standards for cloud native computing.
 - Different projects are hosted by the CNCF.
 - Of these, Kubernetes is a key project, providing a platform for running cloud native applications.
 
##### Ecosystem and Distributions
 - CNCF provides an open environment where different projects can provide solutions for cloud native computing.
 - As a result it has different projects that propose a solution for the same problem.
 - While running Kubernetes, users need to choose which projects to use on top of K8s to provide which type of functionality.
 - This can be done by selecting individual projects and integrating them with Vanilla Kubernetes.
 - As an alternative, a Kubernetes distributions can be used.
 
##### Kubernetes Distributions
 - In Kubernetes distributions, CNCF projects are integrated in a working Kubernetes environment.
 - Some distributions are "opinionated", which means that just one CNCF project is integrated to provide a specific solution.
 - Some distributions are more open, and leave a choice to its users about the specific solution that they're going to use.
 - The distributions may also provide support, which makes it possible to run Kubernetes in environments where reliability is important.
 
##### Distribution Types
 - Cloud Managed Distributions integrate in a public cloud offering, and includes
 1. EKS(Amazon) 2. AKS(Azure) 3. GKE(Google)
 - On premise distributions focus on installation in a company's datacenter on private cloud.
 1. Google Anthos, 2. Rancher 3. Red Hat OpenShift 4. Canonical Kubernetes
 - Minimized Kubernetes distributions are provided for testing and learning
 1. MiniKube 2. K3s 3. OpenShift Local(CRC)
 - The scope of a distribution is not always limited to a specific environment.
 - In this course, MiniKube is the preferred learning environment.

##### 5. Kubernetes Architecture
 - The control plane consists of one or more nodes where the Kubernetes core services are running:
 a. kube-apiserver: provides access to the API
 b. etcd: the Kubernetes database
 c. kube-scheduler: responsible for scheduling pods at a specific location.
 d. kube-controller-manager: manages core Kubernetes processes.
 - The worker node run the containerized applications by using two core services
 a. container runtime: the part that actually runs containers
 b. kubelet: the part that is contacted by the kube-scheduler to run the actual containers in pods.
 
##### 6. Essential API resources
 - Kubernetes API resources allow for storing and running applications in a Kubernetes environment.
 - Essential resources include:
 a. pod : the minimal entity managed by kubernetes. Runs one or more container(s). Pod's important feature is IP address, containers don't have an IP address.
 b. Deployment : adds replication and zero-downtime updates to Pods.
 c. ConfigMap: used to store configuration files and startup parameters.
 d. Services: load balances incoming traffic to application instances.
 e. Ingress/Gateway API: provides a reversed proxy for application access.
 f. Persistent Volumes: represent persistent non-ephemeral storage.
 
The pod is the smallest unit that the kubernetes manage. Within this pod, containers are made to run. Kubernetes may run multiple replicas of the pod, called Deployment. Deployment is used to run scalable applications.

Other resources that can be used are, `ConfigMaps` storing the configurations, `Secret` used to decode the ecrypted configurations, `service` can be used as a load balancer for multiple instances of pods, ingress is a reverse http proxy.

 - documentation is available in the exam.

##### cmds
 - minikube status
 - kubectl api-resources -> lists all the resource
 - kubectl explain <resourceName> -> the doc of the resource
 
#### Creating Lab Environment
1. Kubernetes Deployment Options
2. MiniKube Installation and working
3. Running Kubernetes in cloud
4. Building an All-in-one kubernetes Distributions
5. Running your first application

##### 1. Kubernetes Deployment Options
 - To learn Kubernetes, you can use any Kubernetes environment.
 - To prepare for CKAD in a fully functional environment where you don't have to lose time finding out how stuff works, Minikube is the best option.
 - Some demo in the course work smoothly in on Minikube, but may requrie additional configuration on other distributions.
 - Minikube can be installed on any operating system.
 - As in the exam, you"ll be working with kubernetes from an Ubuntu Linux system. It's recommended to install Minikube on top of an Ubuntu Virtual Machine.
 - WSL is also supported.
 
 - When a cloud provider is ensuring that all the requirements(like Controller, Worker nodes etc..) are setup for you, we call this managed kubernetes.
 - The AIO(all in one) model, is minikube, it has all the requirements in it(controller nodes, worker nodes etc).
 
##### 2. MiniKube
 - Install it on Linux.
 
##### 3. Kubernetes in cloud
 - Kubernetes can be used in all public cloud platforms.
 - Specific functionality may be taken care of by the public cloud provider.
 a. Handling incoming traffic through ingress
 b. Limiting incoming traffic with NetworkPolicies
 c. Storage allocation
 - This makes managed public cloud solutions sub-optimal for learning, and therefore they are not recommended for preparing CKAD.
 
##### 4. Building an All-in-One Kubernetes Distributions
