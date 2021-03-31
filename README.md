# cilium-testings

## Objectives

This project is about investigating how Cilium works and see how to integrate it with Junos cRPD.

## Testing Infrastructure

The test infrastructure is a basic server with 2*10 CPUs/256G mem on which a set of Virtual Machines is deployed.

Basic node provisionning with ubuntu focal
```
root@5b11s15:~# cat /etc/os-release 
NAME="Ubuntu"
VERSION="20.04.1 LTS (Focal Fossa)"
ID=ubuntu
```

Install multipass via snap for Virtual Machine Instantiation

```
apt install snapd -y
snap install multipass
```

Launch 3 Ubuntu VMs 

```
multipass launch -n ubuntu1 --cpus 6 --mem 8G --disk 30G
multipass launch -n ubuntu2 --cpus 6 --mem 8G --disk 30G
multipass launch -n ubuntu3 --cpus 6 --mem 8G --disk 30G
 ```

 Check that it all worked out

```
root@5b11s15:~# multipass list                                                  
Name                    State             IPv4             Image
ubuntu1                 Running           10.81.127.54     Ubuntu 20.04 LTS
ubuntu2                 Running           10.81.127.104    Ubuntu 20.04 LTS
ubuntu3                 Running           10.81.127 233    Ubuntu 20.04 LTS
```

Microk8s Install 

```
root@5b11s15:~# multipass shell ubuntu2
[...]
ubuntu@ubuntu2:~$  sudo snap install microk8s --classic --channel=1.19
Fetch and check assertions for snap "core" (10859)                                            |
Mount snap "core" (10859)                                                                     /
Mount snap "core" (10859)                                                                     \
Mount snap "core" (10859)


microk8s (1.19/stable) v1.19.7 from Canonicalâœ“ installed

ubuntu@ubuntu2:~$ sudo usermod -a -G microk8s $USER
ubuntu@ubuntu2:~$ sudo chown -f -R $USER ~/.kube
ubuntu@ubuntu2:~$ exit

root@5b11s15:~# multipass shell ubuntu2
[...]
```
