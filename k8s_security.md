## Supply Chain
* What will you use as the base image to build your containers?
    * Best practice is to use a container with the most minimal image possible to prevent adding unnecessary security surface area
    * Alpine is the opensource standard (5MB) but Red Hat makes a Universal Base Image (ubi8-minimal 38MB) as well
* What tool will you use as a local container registry for development environment? For production environment?
    * Kubernetes needs a place store and retrieve container images during development and deployment
    * Quay is Red Hat’s product, but there are many other choices as well
* What tool will you use to scan container images for vulnerabilities and how will that process be automated?
    * Best practice is to scan container images as a test at the end of the build step and to continuously scan the container registry
* What tool will you use to build container images?
    * This could be the same tool as the container runtime (such as Docker or Podman) or a separate tool that just an automated part of the CI/CD pipeline (such as Buildah)
    * Best practice is to have the build, scan, store process all be automated when a developer checks the code into source control
* What tool will you use to deploy containers to running Kubernetes clusters?
    * Best practice is to use an ImagePolicyWebhook to reject images that don’t meet security scan policies
    * Some tools that support this are Flux, Argo CD, Jenkins X
## Containers
* Which container runtime will you use?
    * Docker requires a continuously running daemon, Podman does not
    * Both must be specifically configured to run in “rootless” mode or the container will be run on the host with root privileges  
    * If you use Docker, don’t mount the Docker daemon’s socket inside containers, since this would allow a process within the container to execute commands that give it full control of the host. Also, an SELinux profile can be created to prevent containers from accessing the socket
* How will you prevent containers from acquiring new privileges? 
    * Best practice is to set the no_new_priv bit and possibly remove setuid and setgid permissions from images
    * If an image needs privileges, best practice is to set specific linux capabilities and bind them to specific ports on the container
* Will containers have a specific user defined in the image, and will you enable user namespace support to map container user to host user? 
    * Best practice is to avoid sharing host’s network namespace, process namespace, IPC namespace, user namespace, or UTS namespace, unless necessary, to ensure proper isolation between containers and the underlying host
    * Recommend setting the production containers’ root filesystems to read-only because they shouldn’t ever need to be altered while running, only replaced by new running containers
* Containers typically don’t have SSH by default, and it’s a good practice to NOT run SSH, if possible
## Kubernetes
* By default, Kubernetes is configured very insecurely as essentially a flat network with all containers have equal access to host CPU and memory resources. Significant configuration is required to operate securely. Plan to specify exact memory and CPU needs for each container.
* What tool will you use to avoid hardcoding secrets and, instead, inject them into containers at runtime?
    * You should also ensure that secrets are not being passed as environment variables but are instead mounted into read-only volumes in your containers at runtime. Tools that do this are Hasicorp Vault and Cyberark Conjur
    * Have you determined a Kubernetes namespace schema to ensure resources are isolated have least privilege possible?
* What tool will you use to enforce Kubernetes network policies within each namespace, ingress and egress? 
    * Examples are Calico, Flannel, Canal, Weave Net, and Traefik
* What RBAC schema do you intend to use for users, groups, and cluster-amin privileges?
* Plan to create SELinux rules on CCE host to prevent random containers (not ones from our registry) from mounting to our container’s socket and taking control
* How will you use PodSecurityPolicy’s CAP DROP and CAP ADD features to specify only the needed linux capabilities?
* What tools will you use to scan running cluster endpoints for policy compliance? 
    * Examples are Kyverno, Kube-bench, Kube-hunter, and Twistlock
* What performance monitoring and logging tools will you use?
    * Examples are Prometheus, Grafana, Loki, Trafeik
* Here is a list of random best practices to be mindful of when configuring Kubernetes:
    * Impose per container PID limits
    * Don’t configure your mount propagation rules as shared.
    * Container mount propagation mode should be set to private, if possible
    * Don’t use the default bridge “docker0.” Using the default bridge leaves you open to ARP spoofing and MAC flooding attacks. Instead, containers should be on a user-defined network and not the default “docker0” bridge.
    * If the containers require volume mounts, they should be mounted read-only to prevent the container from altering the host OS, if possible
    * Ensure kubelet configured with the --anonymous-auth=false flag
    * Pay special attention to the Kubernetes API server and kube-scheduler configurations
    * Ensure the configuration files on the master node have read/write permissions to prevent unauthorized access 

