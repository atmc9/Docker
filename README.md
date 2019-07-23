# Docker Architecture and Theory

1. Container: An Isolation area(NameSpaces) of OS with resource usage limits (ControlGroups) applied.   
2. Docker Engine: takes the commands -> [(deamon) -> (containerd) -> (OCI)] NameSpaces and ControlGroups magic
3. NameSpaces: They are about Isolation
4. ControlGroups: Thye are about grouping objects and setting resource limits 
5. Container in Linux: Isolated grouping of namesspaces like 
    * Process ID - pid  
    * Network - net  
    * Filesystem/mount - mnt  
    * Inter-process comms - ipc
    * UTS - uts
    * User - user
6. Union File sysytem: Combine read only or block files system and present as a single unified file sysytem. 

## Docker Engine plugins: 
* Native Orchestration - Swarm
* On-prem Secure Registry
* Universal Control Plane with ops-ui, R-back policies
* Echo sysytem plugins

## History
 * dtCloud - with LXC and AUFS, LXC is use dby docker but later created libcontainer 
 * Initially Docker demon has all the code for creating containers and become very monolithic. 
 * In 2016 Docker refecatored the code and implemneted Open Container Initiativ OCI(Image, RunTime Spec) standard. 
 * Having OCI and ContainerD decoupled from Demon, we can easily upgrade Docker in prod by not effecting containers. 
 
## Windows:
 * In windows we will have Client(docker.exe), Demon(dockerd.exe)  
 * Compute Services layer instead of (ContainerD,OCI)
 * In linux we have single process for container
 * Windows has inter-dependency of processes, so when we start windows container it will start the service SMSS that checks all reuired processes are ready - like init in linux
 * Types of contianers on windows: 
    * Native Windows conatiners: Namespace for isolation, all shares the Host OS Kernal
    * Hyper-V isolation container: Windows spins a light weight Hyper-V VM and it used it separate OS
    * only one container for Hyper-V VM is possible
    
 # Docker Images
 
 1. Image: is a read only template for application containers. It contains all code and supporting files to run application. (OS files app files, Manifest)
 2. Images are build time constructs, containers are runtime
 3. Image is a stack of layers
 4. We store Images in Registry, on-prem or Cloud
 5. How to make changes to Image as it is read only, for every container we will ahve a thin writable layer.
 6. Pull Images ->
      * Get Manifest
         * Get the fat Manifest and look for image manifest entry for your OS Type, Architecture
      * Get Layers: Get them from registry's blob store
      * all layers can be verified in lynux at : /var/lib/docker/aufs[diver]/diff
 
 ## Docker Image Commands   
(redis -> RepoName;  )
* **Pull image from registry**: docker image pull redis   (default assumes from docker-hub registry)
* **Look all local images**: docker image ls
* **Image history**: docker history redis 
* **Image Json Config**: docker image inspect redis  
* **Image Delete**: docker image rm redis

## Docker Registries (where images live in)

NOTE: docker.io/redis:latest  [REGISTRY/REPO:IMAGE(tag)]

1. Default registry -> Docker hub
2. DTR -> Docker Trusted Registry (on-premis registry comes with enterprise version)
3. all images pulled will get into logal registry 
    * linux -> /var/lib/docker/overlay2<Storage-driver>
    * windows ->  C:\programData\docker\windowsfilter
4. Official, unofficial images. Official images lives at top level of Hub namespace
    * docker.io/redis , docker.io/nginx:1.13.5
5. On wire layers are compressed and its distributed hash(compressed hash) is included in manifest

## Docker Images best practices:
  * Use official images
  * Look for small size images
  * use alphine image if we dont have image what we need
  * use explicit image reference

# Containerizing an APP

 * Dockerfile is used to create image
 * create an image -> docker image build -t imagename .  [. is current working directory is provided as context]
 * git remote repo can also be passed for image build context
 * during image creation multiple intermediate containers got created and removed.
 
 ## Dockerfile 
   * instructions for building images
   * INSTRUCTION <value>
   * FROM always first instruction
   * FROM = base image
   * Good practice to list maintainer
   * RUN = execute command and create layer
   * COPY = copy code into image as new layer
   * some instrcutions add metadata instead of layers
   * ENTRYPOINT = default app for image/container
   * Multi-stage docker file gives a leaner way to build images for production
   
# Docker Container

* atomic unit of scheduling in docker work is container, in virtulaization it is VM, in kubernetes it is pod
* an contianer is an runtime of image, with just a thin writable container layer added
* each container generally runs a single process and has a single job
* microservices is the way to go using dockers, keep a container as simple as possible and use API's to communicate 
* dockers can very well be used for running traditional apps
* starting, stopping a container does not destroys its data
* every image has a default process to run (sh,..) can be seen by inspecting image
* default process for new containers -> CMD: Run time arguments override CMD instructions -> ENTRYPOINT: Run-time arguments are appended to ENTRYPOINT

## Docker Container Commands:

* docker container run -it(interative terminal) alpine(IMAGE) sh(which process to run)
* ctrl + P + Q to get out of terminal (does not kills the process of shell)
* docker container stop id(few chars of start)
* docker container ls -a  -> lists all the containers even the stopes one
* docker container start id
* docker container exec -it 60 sh  -> takes us to shell commands
* docker container rm $(docker container ls -aq) -f   -> forcely removing all containers 
* docker port containername   -> lists the port mapping between host and container
* docker inspect -f "{{ .NetworkSettings.Networks.nat.IPAddress }}" conatinername  -> gets the ip address of container

## Logging:

* 2 types of logs engine/daemon, container logs(app logs)
* engine/demon
     * Linux: 
         * systemd: journalctl -u docker.service
         * non-systemd: try  /var/log/messages
      * Windows:
         * ~/AppData/Local/Docker
 * container logs
   * default support STDOUT STDERR
   * set default logging driver in daemon.json, any new containers will use this driver
   * override per-container with   --log-driver --log-opts
   



 
 
 
 
 
 
 
 
 
 
 
 
 
 
