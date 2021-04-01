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
multipass shell ubuntu1
sudo snap install microk8s --classic --channel=1.19

```
Result:
```console
root@5b11s15:~# multipass shell ubuntu1
sudo snap install microk8s --classic --channel=1.19Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-70-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Apr  1 02:22:21 PDT 2021

  System load:  0.0               Processes:             151
  Usage of /:   4.5% of 28.90GB   Users logged in:       0
  Memory usage: 3%                IPv4 address for ens4: 10.101.169.212
  Swap usage:   0%

 * Introducing self-healing high availability clusters in MicroK8s.
   Simple, hardened, Kubernetes for production, from RaspberryPi to DC.

     https://microk8s.io/high-availability

1 update can be installed immediately.
0 of these updates are security updates.
To see these additional updates run: apt list --upgradable


sudo snap install microk8s --classic --channel=1.19To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@ubuntu1:~$ sudo snap install microk8s --classic --channel=1.19
Download snap "core" (10908) from channel "stable"                 

microk8s (1.19/stable) v1.19.7 from Canonicalâœ“ installed
```
Configure permissions
```
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
exit
multipass shell ubuntu2
```
