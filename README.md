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
* Eco-sysytem plugins

## History
 * dtCloud - with LXC and AUFS, LXC is use dby docker but later created libcontainer 
 * Initially Docker demon has all the code for creating containers and become very monolithic. 
 * In 2016 Docker refecatored the code and implemneted Open Container Initiativ OCI(Image, RunTime Spec) standard. 
 * Having OCI and ContainerD decoupled from Demon, we can easily upgrade Docker in prod by not effecting containers. 
 
## Windows:
 * In windows we will have Client(docker.exe), Demon(dockerd.exe)  
 * Compute Services layer instead of (ContainerD,OCI)
 * In linux we have single process for container
 * Windows has inter-dependency of processes, so when we start windows container it will start the service SMSS that checks all required processes are ready - like init in linux
 * Types of contianers on windows: 
    * Native Windows conatiners: Namespace for isolation, all shares the Host OS Kernal
    * Hyper-V isolation container: Windows spins a light weight Hyper-V VM and it used it separate OS
    * only one container for Hyper-V VM is possible
    
 # Docker Images
 
 1. Image: is a read only template for application containers. It contains all code and supporting files to run application. (OS files app files, Manifest)
 2. Images are build time constructs, containers are runtime
 3. Image is a stack of layers
 4. We store Images in Registry, on-prem or Cloud
 5. How to make changes to Image as it is read only, for every container we will have a thin writable layer.
 6. Pull Images ->
      * Get Manifest
         * Get the fat Manifest and look for image manifest entry for your OS Type, Architecture
      * Get Layers: Get them from registry's blob store
      * all layers can be verified in linux at : /var/lib/docker/aufs[diver]/diff
 
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
3. all images pulled will get into local registry 
    * linux -> /var/lib/docker/overlay2<Storage-driver>
    * windows ->  C:\programData\docker\windowsfilter
4. Official, unofficial images. Official images lives at top level of Hub namespace
    * docker.io/redis , docker.io/nginx:1.13.5
5. On wire, layers are compressed and its distributed hash(compressed hash) is included in manifest

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

* atomic unit of scheduling in docker world is container, in virtulaization it is VM, in kubernetes it is pod
* an contianer is an runtime of image, with just a thin writable container layer added
* each container generally runs a single process and has a single job
* microservices is the way to go using dockers, keep a container as simple as possible and use API's to communicate 
* dockers can very well be used for running traditional apps
* starting, stopping a container does not destroys its data
* every image has a default process to run (sh,..) can be seen by inspecting image
* default process for new containers -> CMD: Run time arguments override CMD instructions -> ENTRYPOINT: Run-time arguments are appended to ENTRYPOINT

## Docker Container Commands:

* docker container run -it(interative terminal) alpine(IMAGE) sh(which process to run)
* docker container run --rm -d alpine sleep 1d   (creates and run the container in background)
* ctrl + P + Q to get out of terminal (does not kills the process of shell)
* docker container stop id(few chars of start)
* docker container ls -a  -> lists all the containers even the stopes one
* docker container start id
* docker container exec -it 60 sh  -> takes us to shell commands inside that container
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
 * container logs: (takes standout, stderr steams and forwards them to somewhere else syslog, gelf, splunk, fluentd), it is all get done using logging-drivers, configured at deamon.json
   * default support STDOUT STDERR
   * set default logging driver in daemon.json, any new containers will use this driver
   * override per-container with   --log-driver --log-opts
   * by default docker hosts use json logging, so we can check log files using -> docker logs <container>
   * splunk can be configured as logging solution
   
# Docker Swarm : secure cluster of docker nodes
 * swarm has 2 parts, secure cluster(key feature), Orchestrator (might be given to kubernetes later)
 * Secure Swarm Cluster:
      * Cluster of Docker nodes with managers, workers,
      * mutual tls authentication 
      * network chat is encrypted
      * cluster store is encrypted
 * Native swarm or kubernetes can be used on Docker swarm
 * swarmkit is one powers swarm mode
 * after swarmkit is intergarted in docker from v1.12+, docker has 
      * sigle- engine mode: work with each node separately
      * swarm mode: work in a cluster mode 
 * raft consensus group handles all distributed consensus stuff, like electing a new manager leader   
 * have odd number of managers to acheive quorum 
 * connect your managers in reliable networks (if in aws dont keep them across regions)
 * when a worker join, it wont get access to cluster store, but gets full list of ips for managers, all get their certificates
 * manager,worker token ->  SWMTKN-1-[ClusterIdentifier(hash of cluster certificate)][ID -determines if I am a worker or manager]
 * restarting a manager, restoring a old backup causes problem, so docker does lock a swarm called autolock
         * prevents restarted Managers from automatically rejoining the swarm
         * loading the encryption keys and decypting the raft logs 
         * prevents accidentally restoring the old copies of swarm 
 * Orchestration: we can deploy application on swarm cluster (and declare 4 containers to run), if a node break, swarm and kubernetes will observer cluster and if desired state changes, it will self heal (adding a new container), balancing the load, rolling updates  
 
## Docker Swarm commands: 
 * docker swarm init -> covert a docker node in a swarm mode 
         * single node becomes mananger, leader, 
         * certificate authority, 
         * issues client certificate, 
         * creates cluster store
         * default certification rotation policy, 
         * creates cryptographic keys to join as managers, workers
 * docker swarm join (crypto join token for manangers) -> to join as a manager in a swarm
 * docker node ls -> to view all docker nodes both managers, workers 
 * docker swarm join-token manager(or worker)  -> this will gives the command to join a swarm
 * docker swarm join-token rotate manager -> to rotate the token if the key is compromised
 * sudo openssl x509 -in /var/lib/docker/swarm/certificates/swarm-node.crt -text   -> look at client certificate on a node 
      * in the subject: O-(Organization - SwarmID), OU(OrganizationUnit - node's Role), CN(Canonical role - cryptographic node ID) 
 * docker swarm init --autolock  -> Autolock new swarm (autolock is not default enabled)
 * docker swarm update --autolock=true -> Autolock existing swarm
 * sudo service docker restart
 * docker swarm unlock      provide the key -> this allows to join the swarm
 * docker swarm update --cert-expiry 48h    -> can be verified using docker system info
 
# Container Networking

* Containers may need to talk to VM's, internet or vice-versa
* Bridge Networking (single-house networking) - oldest and crappiest
* built-in network on docker -> bridge(linux), nat(windows) - it is reffered as docker0
* to have communication across networks we need port mappings
* overlay-networks (multi-host networks)  -> communicating across containers over multiple network
* layer-2 network spanning multiple hosts
* docker overlay encrypts control, data planes
* to communiate to VMs MACVLAN (linux), transparent(windows)
* using MACVLAN conatiners get ip and mac address makes containers first class citizans
* MACVLAN require promiscuous mode on host nick, which cloud providers wont allow mostly
* IPVLAN solves that, but not so standard solution
* when a container is added, it will be added to default bridge network
* For overlays, we need swarm mode

## docker network commands

* docker network create  -o encrypted  -> creates a default network, bridge driver
* docker network ls   -> view all networks
* docker network inspect bridge   -> inpect a network
* docker container run --rm -d --name web -p 8080:80 nginx   -> port mapping, added to default network
* docker network create -d bridge(driver) golden-gate(bridgename) 
* docker container run --rm -d --network golden-gate alpine sleep 1d  -> creating container in a specific network
* docker network create -d overlay overnet  -> this overlay network is now scope to swarm
* docker service create -d --name pinger --replicas 2 --network overnet alpine sleep 1d   -> we will get service with 2 replicas
      * docker service create -> creating new native swarm service
      *  --name pinger        -> calling it pinger
      *  --replicas 2         -> creating 2 taks of replicas
      *  --network overnet    -> putting it on new overlay
      *  alphine sleep 1d     -> telling what to run
* docker service ls   -> to list the services
* docker service ps pinger   -> lists the task of service
* docker service rm $(docker service ls -q)

## network services:
* Service Discovery: using the name registered every container in network able to ping it using name
   * every service gets a name
   * names are registered with swarm DNS
   * every container in a service gets DNS Resolver that forwards lookups to Swarm based DNS service. 
   * all swarm services are pingable by name
* Load Balancing: makes every node in the swarm know about every service
   * 1. ingress loadbalancing: any node in swarm will be able to discover the relica for the service 
   * say my swarm contains -> 3 nodes 
   * docker service create -d name web --replicas 1 --network overnet -p 8080:80 nginx   -> service got ceated with 1 replica 
   * say the service container got created on node1
   * docker service inspect web --pretty   -> check the network port 
   * if I access the site using node2 ip address, still the site works 

# volumes and Persistent data

* docker volume create   -> creates a persistent volumes that can be independent from containers
* docker container run -dit --name voltest  --mount source=ubervol,target=/vol alpine:latest
* we can also mount the volumns on external high end sysytems like SAN, NAS using docker storage drivers
* volumes live on host ->   /var/lib/docker/volumes/
 
# Docker secret

* 
 
 
 
 
 
 
 
 
 
 
