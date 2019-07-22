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
    
