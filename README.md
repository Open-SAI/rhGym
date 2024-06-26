# Red Hat Gym 
A pragmatic space to save notes, tips, commands in a Red Hat Engineer training journey...

### Git / github 
- ```$ git init``` &rarr; Init local new repo 
- ```$ git checkout -b newBranchName``` &rarr; create a new branch and check it out
- CONTRIBUTING.md file ( /.github, root, /docs) &rarr; recommended to explain the contribution policy for the project
- There is templates to new issues (.github/ISSUE_TEMPLATE.md) and pull request (.github/PULL_REQUEST_TEMPLATE.md)
- ```git push --set-upstream origin <BRANCH_NAME>``` &rarr; In a forked repository, in the process to make changes in a repository not owned by us, it creates a new branch on the upstream repository (remote repository linked to our local repository) on our github fork, and pushes all our commits. Before this step, can be possible to make a "Compare & pull request" in Github.

### Basics
- A container is an encapsulated process with his runtime dependencies (independents libraries from the host operating system)
- Container image layers are immutable
- The container engine add a writable layer for runtime file modifications
- Container are ephemeral (writable layer disappears in the container removing)
- Open Container Initiative (OCI) is the governance organization to standards in the containers world

### Podman  
Podman: tool to pods, containers and images management:
- ```podman -v``` &rarr; podman version
- ```podman pull registryUrl/user-Org/repository:imgTag``` &rarr; pull remote image
    - Registries:
        - Quai.io
            - can be used to store custom images
        - Red Hat Registry
            - access.redhat (No auth)
            - redhat.io (auth)
        - Docker Hub
        - Amazon ECR
    - https://catalog.redhat.com &rarr; container images catalog
- UBI &rarr; Universal Base Image, common container images...
- /etc/containers/registries.conf &rarr; podman containers registries
- skopeo &rarr; command line tool for working with container images (inspect, copy, sign, convert)
- ```podman login registryUrl``` &rarr; authenticate to remote registry
- ```podman search imageName``` &rarr; search image in enabled registries
- ```podman image ls / podman images``` &rarr; list the images available locally 
- ```podman build --file ContainerFILE --tag registrie/user/imageName/TAG``` &rarr; build an image, can be later pushed to registry
- ```podman build -f ContainerFILE -t ContainerImage:TAG``` &rarr; create an image named ContainerImage:TAG (-t option), from the ContainerFILE descriptor
- ```podman build ContainerImage:TAG .``` &rarr; create an image named ContainerImage:TAG using the local Containerfile in "." (note the default name "Containerfile")
- ```podman push registry/user/imageName/TAG``` &rarr; push an image to registry
- ```podman image inspect registryUrl|containerNameId``` &rarr; shows information about the image
    - --format="{{.Key.Subkey}}" &rarr; show specific part of the inspect information
- ```podman login registryUrl``` &rarr; authenticate to remote registry
- ```podman exec -it Container COMMAND``` &rarr; execute the COMMAND in the Container
- ```podman image rm Registry/NameSpace/ImageName:TAG``` &rarr; delete local images (maybe needed to stop the containers that are running this image: ```podman stop containerName```), the -f option force the remove, ```rmi``` can be used instead ```image rm```
- ```podman image prune``` &rarr; clean dangling images
- ```podman image prune -a``` &rarr; clean dangling and unused images (use -f option to force and avoid interactive prompt)
- ```podman export -o fileBackup.tar CONTAINER_NAME|CONTAINER_ID``` &rarr; export the container file system (create a snapshot to restore the container)
- ```podman import fileBackup.tar NAME_CONTAINER:TAG``` &rarr; import the .tar file to restore the container file system
- ```podman run -d --rm --name ContainerName -p 3000:3000 ContainerImage:TAG``` &rarr; run an instance of ContainerImage:TAG in background (-d), named ContainerName, with a port mapping 3000:3000, when the container is stopped, will be removed
- ```podman run -d -p 8080:8000 --name TheContainer ContainerImage``` &rarr; run an instance of ContainerImage in background (-d), named TheContainer, with a port mapping from the Port Host 8080 to Container Host 8000
- ```podman stop ContainerName``` &rarr; the container ContainerName will be stopped
- ```podman image tag ContainerImage:0.1``` &rarr; add a tag to the ContainerImage, the IMAGE ID remains the same value because the tag point to the same image
- ```podman image rm ContainerImage``` &rarr; remove the ContainerImage, to delete an image it is necesary stop and delete all the containers that use the image (-f option to force)
- ```podman push Registry/Url/user/ContainerImage:tag``` &rarr; push the ContainerImage to the Registry
- ```podman image tree ContainerImage:tag``` &rarr; inspect the image layers in the ContainerImage:tag

#### Images
- Containerfile &rarr; set of instructions to build a container image
    - each instruction create a new layer in the resulting image (layers stacked in the resulting container image)
    - podman supports Dockerfiles and Containerfiles
    - use a small domain-specific language (DSL)
        - FROM &rarr; set the base image for the new container image
        - WORKDIR &rarr; set the current directory of working
        - COPY & ADD &rarr; ...files from build host to container image WORKDIR path. Relative paths...
        - RUN &rarr; run command in the container, results in a new container layer
        - ENTRYPOINT &rarr; executable to run when the container is started
        - CMD &rarr; command to run when container is started
        - USER &rarr; active user whitin the container (root is not recommended)
        - LABEL &rarr; metadata, key-value... examples: org.opencontainers.image.authors="My Name" // com.example.environment="production"
        - EXPOSE &rarr; metadata, the port of the app/s in the container
        - ENV &rarr; vars available in the container
        - ARG &rarr; built-time variables
        - VOLUME &rarr; where store the data outside of the container, path where mounts volume inside of the container, supports multiple paths to supports multiple volumes, his value is inmutable after his calling in the Containerfile (```podman volume create VOLUME```, ```podman volume ls --format="{{.Mountpoing}}\t{{Name}}"``` &rarr; to list)
        - **Best Practices**
            - to use ENV configurations to store dependent container configurations, thats vars can be called from the apps &rarr; ENV DB_HOST="db.domain.com"
            - with the *--build-arg* flag to use ARG build-time vars to customize the container build &rarr; ```ARG MY_VAR="my_value" RUN command "parameter ${MY_VAR}"``` 
            - with the use of ARG values in the ENV instructions, can be possible to preserve the build-time variables for runtime
            - the option *prune* can be used to remove unused volumes &rarr; ```podman volume prune```
            - ```podman secret create SECRET_NAME SecretFile``` &rarr; create a secret using a file
            - ```podman secret create SECRET_NAME SecretFile``` &rarr; create a secret using a file
            - ```printf "MyS3cr3t" | podman secret create -``` &rarr; create a secret using the standard input
            - ```podman secret create SECRET_NAME SecretFile``` &rarr; create a secret using a file
            - ```podman secret create SECRET_NAME SecretFile``` &rarr; create a secret using a file, the *--driver pass* store the secret in a GPG encrypted file, this option also supports *shell* script
            - ```podman secrets ls``` &rarr; list stored secrets
            - ```podman secrets rm My_Secret``` &rarr; remove My_Secret secret
            - ```podman run -it --secret My_Secret --name MyApp``` &rarr; make My_Secret available for use of container MyApp, mounted in a *tmpfs* container dir */run/secret*. Neither *podman commit // podman export* copy secret data to the image or *.tar* file
            - chaining instructions reduce the number of image layers &rarr; ```RUN command0 params0 && command1 params1 && command2 params2```
            - ```--squash-all``` option squash the layers from the previous or parent image (reduce size of builded images)            
            - using multistage builds reduce image layers (reduce size) &rarr; first stage: ```FROM image/url:TAG **as builder**```, second stage:```FROM another/image/url:TAG \ COPY **--from-builder** Original_Source Dest_Source``` (reuse the first build)
            - podman rebuilds all the layers only if the **package.json** is changed
    - When is not used a tag, podman uses the 'latest'
- In the building of the image, is used a base image (UBI: Universal Base images), the base images determines
    - Init System
    - Package manager
    - Filesystem layout
    - Preinstalled dependencies and runtimes (Python, Node.js, etc...)
    - Processor compatibility...
    - UBI variants 
        - standard (primary UBI)
        - init (support systemd)
        - minimal (microdnf)
        - micro (smallest UBI, does not include package manager)
    - ```podman image tree MyImage``` &rarr; list the tree of the layers in the container image
#### Rootless containers
- containers that do not require root privileges (unprivileged containers)
    - containerized process doesn't use the root user (id=0)
    - root user inside the container is not the root user outside 
    - the container runtime doesn't use the root user
    - podman use cgroup v2 (resources limit), slirp4netns (networking), fuse-overlaysfs (Copy on Write file systems - COW)
    - subordinate ID ranges 
        - allowed ID Ranges configured in the host files: /etc/subuid and /etc/subgid (incluye starting id number and limit)
        - subID ranges can be configured with the usermod command
    - the ```podman top``` command can be used to review id mapping &rarr; ```podman top CONTAINER_NAME huser user``` (a *cat* to */proc/self/uid_map /proc/self/gid_map* can be too used)
    - required use of privileged ports or utilities can be configured with the *sysctl* utility &rarr; ```sysctl -w "net.ipv4.ip_unprivileged_port_start=PRIVILEGED_PORT" ```
#### Volumes
- COW &rarr; Copy-on-write file system
    - every instruction that modifies the container adds a data layer in read-only mode including the set of changes (`diffs`)
    - ```podman image tree NAME-CONTAINER``` &rarr; inspect the image layers
    - in the runtime podman create `thin` read-write additional layer on top of the previous layers
    - when the container is deleted, podman destroy the last read-write layer &rarr; the runtime data is ephemeral!
    - there is a performance bottleneck in write-intensive containerized processes
- Persistent Data (Store data on Host Machine)
    - data is persistent across container deletions.
    - write-intensive without the limitations of the COW file systems
    - data can be shared between multiple containers at runtime and at same time 
    - support the NFS protocol
    - there is volume mounts (managed by podman &rarr; production)
        - ```podman volume create VolumeName``` &rarr; create a new volume
        - ```podman volume inspect VolumeName``` &rarr; inspect a volume
        - ```podman volume import VolumeName data.tar.gz``` &rarr; import data to a volume
        - ```podman volume export VolumeName --output data.tar.gz``` &rarr; export volume data
        - ```podman run -e MYSQL_ADMIN_PASSWORD=passwd --network my-net --mount  type=tmpfs,tmpfs-size=1024M,destination=/var/lib/data registry.url/rhel/db:latest``` &rarr; using a `tmpfs` mounting
        - is not needed review SELinux permissions because podman manage the volumes
    - there is data mounts (managed by the user &rarr; testing)
    - commands:
        - the ```--volume``` or ```-v``` option is used to both (```--volume /path/to/the/host OR VolumeName:/path/in/the/container:OPTIONS```)
        - the ```--mount type=TYPE, source=/path/on/the/host, destination=/path/in/the/container```
            - TYPE:
                - bind (mounts)
                - volume (volume mounts)
                - tmpfs (memory ephemeral mounts)
        - it is supported relative (volumes) and absolute paths(binds)
    - ```podman run -p PORT_H:PORT_C --volume /path_in_host:/path_in_container:Z registry.url``` &rarr; the Z option fix possible SELinux problems
        - ```z``` &rarr; different containers share access to a bind mount
        - ```Z``` &rarr; exclusive access to the container
        - ```podman unshare DIR_COMMAND(ls /p/a/t/h)``` &rarr; it is useful to troubleshooting permissions
    

### OpenShift
- It is a *kubernetes distribution*
- Red Hat Hybrid Cloud Console &rarr; operator data uploaded to inspect cluster recommendations
- ```oc login -u developer -p developer https://api.ocp.domain.tld:6443``` &rarr; access to the OpenShift web console
- ```oc whoami --show-console``` &rarr; return the url web console
- The web console
    - it provides mainly two perspectives (Administrator: admin the cluster, Developer: to running apps)
- ```oc get nodes``` &rarr; view the cluster nodes
- ```oc debug node/name``` &rarr; it opens the console of the debug pod
- ```chroot /host``` &rarr; ...to use the host root systems (binaries)
    - ```crictl ps``` &rarr; display the containers that are running in this system (node machine)
    - ```systemctl status kubelet``` &rarr; status of the kubelet service in the node
    - ```systemctl status crio``` &rarr; status of the CRIO-O service in the node
    - the node run a Red Hat Enterprise Linux CoreOS release
    - the container of the debug pod run a standard RHEL
#### Added features over Kubernetes cluster in OpenShift
- Workflow Integrated for developers &rarr; integrate a built-in container registry to use in CI/CD pipelines with a S2I tool to build container images
- Observability &rarr; logging and monitoring services for the applications and the cluster
- Management of Server &rarr; hosts in a cluster use RHEL CoreOS (inmutable operating system to optimized to run containerized applications) managed from OpenShift unified tools (graphical web console)
- Product Versions
    - OpenShift Platform Plus
        - Advanced Cluster Management
        - Advanced Cluster Security
        - Quay private registry platform
    - OpenShift Container Platform
        - Serveless (Knative)
        - Service Mesh (Istio)
        - Pipelines (Tekton)
        - GitOps (Argo CD)
    - Kubernetes Engine
        - latest Kubernetes version
        - enterprise stability and security hardening
        - Linux Core OS container OS deployments
- Managed OpenShift Services
    - AWS &rarr; ```ROSA``` &rarr; Red Hat OpenShift on AWS
    - Azure &rarr; ```ARO``` &rarr; Azure Red Hat OpenShift
    - IBM Cloud  
    - Google Cloud &rarr; Red Hat OpenShift Dedicated 
#### Kubernetes 
- Kubernetes is a powerful framework for creating server clusters to run containerized applications. Provides a model for defining workloads to run on a cluster.
- OpenShift builds upon Kubernetes to provide a comprehensive and feature-rich platform for containerized applications.
- declarative resource management
    - controllers  &rarr; monitoring the cluster state to fullfill the declarated state of cluster
    - namespaces &rarr; provide the capacity to control the resource access based in roles
- pod &rarr; is the most small unit of a kubenetes containerized app, include one or more container to run in a cluster node and support the workloads
- provides a web console that is not installed by default, only supports token based authentication, it works through a proxy (the only point of access)
##### Features
- Load balancing and Service Discovery  &rarr; it reconfigure the network/dns to performance and reliability
- Horizontal scalling  &rarr; from a monitoring, scale the required pods/nodes to support the required load
- Self-healing  &rarr; restart, monitor, reschedule falling applications at application level or cluster level
- Automated rollout  &rarr; in gradual updates, it is possible make a roll back 
- Secrets and configuration management  &rarr; configurations of secrets (usernames, passwords, etc) isolated of the containers development (does not encrypt secrets)
- Operators support &rarr; predefined kubernetes applications to support specific workloads (by example a database deployment)
##### Architecture
- nodes &rarr; servers of the cluster (virtual or physical). There is two types (recommended to run in different servers, there is clusters with up to 5000 nodes):
    - control plane nodes &rarr; the nodes to operate the cluster itself (coordination, scheduling)
    - compute plane nodes &rarr; run the workloads
#### OpenShift terminology
- Pods &rarr; the same concept from Kubernetes
- Deployments &rarr; the process that support a running application
- Projects &rarr; a namespace (same to Kubernetes) to offer multitenancy for apps
- Routes &rarr; used to expose apps and services outside of the cluster
- Operators &rarr; packaged kubernetes apps to extend cluster functionalities
- CRI-O &rarr; Container Runtime Interface service for OCI
- kubelet &rarr; service running in the nodes managing the container lifecycles and reports to the control plane
- node &rarr; bare metal, virtual, cloud machines that runs the pods
- MachineConfig &rarr; definition of the state, changes, files, services, s.o. updates for the CRI-O and kubelet services (resource). If the MachineConfig changes, MCO execute all the needed changes to get the required state
- MCO &rarr; Machine Config Operator, it maintains the configuration and operating systems of the cluster machines (at cluster-level)
#### OpenShift monitoring





