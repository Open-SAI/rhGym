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
- ```podman build --file ContainerFile --tag registrie/user/imageName/TAG``` &rarr; build an image, can be later pushed to registry
- ```podman push registry/user/imageName/TAG``` &rarr; push an image to registry
- ```podman image inspect registryUrl|containerNameId``` &rarr; shows information about the image
    - --format="{{.Key.Subkey}}" &rarr; show specific part of the inspect information
- ```podman login registryUrl``` &rarr; authenticate to remote registry
- ```podman image rm Registry/NameSpace/ImageName:TAG``` &rarr; delete local images (maybe needed to stop the containers that are running this image: ```podman stop containerName```), the -f option force the remove, ```rmi``` can be used instead ```image rm```
- ```podman image prune``` &rarr; clean dangling images
- ```podman image prune -a``` &rarr; clean dangling and unused images (use -f option to force and avoid interactive prompt)
- ```podman export -o fileBackup.tar CONTAINER_NAME|CONTAINER_ID``` &rarr; export the container file system (create a snapshot to restore the container)
- ```podman import fileBackup.tar NAME_CONTAINER:TAG``` &rarr; import the .tar file to restore the container file system
- ```podman run -d -p 8080:8000 --name TheContainer ContainerImage``` &rarr; run an instance of ContainerImage in background (-d), named TheContainer, with a port mapping from the Port Host 8080 to Container Host 8000
- ```podman build -f ContainerFile -t ContainerImageServer``` &rarr; create an image named ContainerImageServer (-d option), from the ContainerFile descriptor
- ```podman image tag ContainerImage:0.1``` &rarr; add a tag to the ContainerImage, the IMAGE ID remains the same value because the tag point to the same image
- ```podman image rm ContainerImage``` &rarr; remove the ContainerImage, to delete an image it is necesary stop and delete all the containers that use the image (-f option to force)
- ```podman push Registry/Url/user/ContainerImage:tag``` &rarr; push the ContainerImage to the Registry
#### Images
- Containerfile &rarr; set of instructions to build a container image
    - each instruction create a new layer in the resulting image (layers stacked in the resulting container image)
    - podman supports Dockerfiles and Containerfiles
    - use a small domain-specific language (DSL)
        - FROM &rarr; set the base image for the new container image
        - WORKDIR &rarr; set the current directory of working
        - COPY & ADD &rarr; ...files from build host to container image. Relative paths
        - RUN &rarr; run command in the container, results in a new container layer
        - ENTRYPOINT &rarr; executable to run when the container is started
        - CMD &rarr; command to run when container is started
        - USER &rarr; active user whitin the container (root is not recommended)
        - LABEL &rarr; metadata, key-value...
        - EXPOSE &rarr; metadata, the port of the app/s in the container
        - ENV &rarr; vars available in the container
        - ARG &rarr; built-time variables
        - VOLUME &rarr; where store the data outside of the container, path where mounts volume inside of the container
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





### OpenShift
- ```ROSA``` &rarr; Red Hat OpenShift on AWS
- ```ARO``` &rarr; Azure Red Hat OpenShift





