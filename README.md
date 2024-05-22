# Red Hat Gym 
A pragmatic space to save notes, tips, commands in a Red Hat Engineer training journey...

### Git / github 
- Init local new repo ```$ git init```

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
- ```podman login registryUrl``` &rarr; authenticate to remote registry

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

### OpenShift
- ```ROSA``` &rarr; Red Hat OpenShift on AWS
- ```ARO``` &rarr; Azure Red Hat OpenShift





