# Cilium install over microk8s 

## Objectives

This objective is simply to have a look at Cilium by installing it in a microk8s VM.

## Testing Infrastructure

The test infrastructure is a basic server with 2*10 CPUs/256G mem on which a set of Virtual Machines is deployed.

Basic node provisionning with ubuntu focal
```console
root@5b11s15:~# cat /etc/os-release 
NAME="Ubuntu"
VERSION="20.04.1 LTS (Focal Fossa)"
ID=ubuntu
```

Install multipass via snap for Virtual Machine Instantiation

```command
apt install snapd -y
snap install multipass
```

Launch an Ubuntu VM 

```bash
multipass launch -n ubuntu1 --cpus 6 --mem 8G --disk 30G
 ```

 Check that it all worked out

```console
root@5b11s15:~# multipass list                                                  
Name                    State             IPv4             Image
ubuntu1                 Running           10.81.127.54     Ubuntu 20.04 LTS
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

sudo snap install microk8s --classic --channel=1.19To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@ubuntu1:~$ sudo snap install microk8s --classic --channel=1.19
Setup snap "microk8s" (2060) security profiles                                                                                       |
Setup snap "microk8s" (2060) security profiles                                                                                       -
Setup snap "microk8s" (2060) security profiles                                                                                       /

Run install hook of "microk8s" snap if present                                                                                       |

microk8s (1.19/stable) v1.19.8 from Canonicalâœ“ installed
ubuntu@ubuntu1:~$ 
```
Tune microk8s kubectl with k alias and autocompletion / Configure permissions 
```
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
echo "alias k='microk8s kubectl'" >>~/.bashrc
echo "complete -F __start_kubectl k" >>~/.bashrc
sudo bash -c "microk8s kubectl completion bash >/etc/bash_completion.d/kubectl"
exit
multipass shell ubuntu1
```

To enable Cilium, just type the following command

```
microk8s enable cilium
```
After the sequence below is finished, you're done. This is super simple.
```console
ubuntu@ubuntu1:~$ microk8s enable cilium
Enabling Helm 3
Fetching helm version v3.0.2.
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 11.5M  100 11.5M    0     0  10.7M      0  0:00:01  0:00:01 --:--:-- 10.7M
Helm 3 is enabled
Restarting kube-apiserver
Restarting kubelet
Enabling Cilium
Fetching cilium version v1.6.
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   119  100   119    0     0    207      0 --:--:-- --:--:-- --:--:--   207
100 21.1M    0 21.1M    0     0  5460k      0 --:--:--  0:00:03 --:--:-- 5588k
Deploying /var/snap/microk8s/2060/actions/cilium.yaml. This may take several minutes.
configmap/cilium-config created
serviceaccount/cilium created
serviceaccount/cilium-operator created
clusterrole.rbac.authorization.k8s.io/cilium created
clusterrole.rbac.authorization.k8s.io/cilium-operator created
clusterrolebinding.rbac.authorization.k8s.io/cilium created
clusterrolebinding.rbac.authorization.k8s.io/cilium-operator created
daemonset.apps/cilium created
deployment.apps/cilium-operator created
Waiting for daemon set "cilium" rollout to finish: 0 out of 1 new pods have been updated...
Waiting for daemon set "cilium" rollout to finish: 0 of 1 updated pods are available...
Waiting for daemon set "cilium" rollout to finish: 0 of 1 updated pods are available...
daemon set "cilium" successfully rolled out
configmap "calico-config" deleted
Warning: apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
customresourcedefinition.apiextensions.k8s.io "bgpconfigurations.crd.projectcalico.org" deleted
customresourcedefinition.apiextensions.k8s.io "bgppeers.crd.projectcalico.org" deleted
customresourcedefinition.apiextensions.k8s.io "blockaffinities.crd.projectcalico.org" deleted
customresourcedefinition.apiextensions.k8s.io "clusterinformations.crd.projectcalico.org" deleted
customresourcedefinition.apiextensions.k8s.io "felixconfigurations.crd.projectcalico.org" deleted
customresourcedefinition.apiextensions.k8s.io "globalnetworkpolicies.crd.projectcalico.org" deleted
customresourcedefinition.apiextensions.k8s.io "globalnetworksets.crd.projectcalico.org" deleted
customresourcedefinition.apiextensions.k8s.io "hostendpoints.crd.projectcalico.org" deleted
customresourcedefinition.apiextensions.k8s.io "ipamblocks.crd.projectcalico.org" deleted
customresourcedefinition.apiextensions.k8s.io "ipamconfigs.crd.projectcalico.org" deleted
customresourcedefinition.apiextensions.k8s.io "ipamhandles.crd.projectcalico.org" deleted
customresourcedefinition.apiextensions.k8s.io "ippools.crd.projectcalico.org" deleted
customresourcedefinition.apiextensions.k8s.io "networkpolicies.crd.projectcalico.org" deleted
customresourcedefinition.apiextensions.k8s.io "networksets.crd.projectcalico.org" deleted
clusterrole.rbac.authorization.k8s.io "calico-kube-controllers" deleted
clusterrolebinding.rbac.authorization.k8s.io "calico-kube-controllers" deleted
clusterrole.rbac.authorization.k8s.io "calico-node" deleted
clusterrolebinding.rbac.authorization.k8s.io "calico-node" deleted
daemonset.apps "calico-node" deleted
serviceaccount "calico-node" deleted
deployment.apps "calico-kube-controllers" deleted
serviceaccount "calico-kube-controllers" deleted
Cilium is enabled
ubuntu@ubuntu1:~$ 
```

You can check the pods that have been created as well as the new crds
```
ubuntu@ubuntu1:~$ k get pods -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   cilium-operator-774f85cdd8-t2mcj   1/1     Running   1          12m
kube-system   cilium-m66m7                       1/1     Running   1          12m
ubuntu@ubuntu1:~$ k get crds
NAME                              CREATED AT
ciliumnetworkpolicies.cilium.io   2021-04-01T10:54:36Z
ciliumendpoints.cilium.io         2021-04-01T10:54:37Z
ciliumnodes.cilium.io             2021-04-01T10:54:38Z
ciliumidentities.cilium.io        2021-04-01T10:54:39Z
ubuntu@ubuntu1:~$ 
ubuntu@ubuntu1:~$ k create deployment alpine --image=alpine --replicas=2
deployment.apps/alpine created
ubuntu@ubuntu1:~$ k get pods
NAME                      READY   STATUS             RESTARTS   AGE
alpine-6b967c77f7-22w5j   0/1     CrashLoopBackOff   4          2m26s
alpine-6b967c77f7-568sf   0/1     CrashLoopBackOff   4          2m26s
ubuntu@ubuntu1:~$ ^
```
Damn, pod creation failed, let's see what describe says.

```console
ubuntu@ubuntu1:~$ k describe pod
Name:         alpine-6b967c77f7-568sf
Namespace:    default
[...]
Events:
  Type     Reason             Age                  From               Message
  ----     ------             ----                 ----               -------
  Normal   Scheduled          117s                 default-scheduler  Successfully assigned default/alpine-6b967c77f7-568sf to ubuntu1
  Normal   Pulled             110s                 kubelet            Successfully pulled image "alpine" in 4.225047254s
  Normal   Pulled             106s                 kubelet            Successfully pulled image "alpine" in 1.777492466s
  Normal   Pulling            92s (x3 over 115s)   kubelet            Pulling image "alpine"
  Normal   Pulled             90s                  kubelet            Successfully pulled image "alpine" in 1.181367686s
  Normal   Created            90s (x3 over 109s)   kubelet            Created container alpine
  Normal   Started            89s (x3 over 109s)   kubelet            Started container alpine
  Warning  BackOff            88s (x3 over 104s)   kubelet            Back-off restarting failed container
  Warning  MissingClusterDNS  76s (x10 over 116s)  kubelet            pod: "alpine-6b967c77f7-568sf_default(a19f226f-1b15-49b6-b28b-8b3a3bc05a5e)". kubelet does not have ClusterDNS IP configured and cannot create Pod using "ClusterFirst" policy. Falling back to "Default" policy.
ubuntu@ubuntu1:~$ microk8s enable dns
Enabling DNS
Applying manifest
serviceaccount/coredns created
configmap/coredns created
deployment.apps/coredns created
service/kube-dns created
clusterrole.rbac.authorization.k8s.io/coredns created
clusterrolebinding.rbac.authorization.k8s.io/coredns created
Restarting kubelet
DNS is enabled

ubuntu@ubuntu1:~$ k get pods -n kube-system
NAME                               READY   STATUS    RESTARTS   AGE
cilium-operator-774f85cdd8-t2mcj   1/1     Running   1          20m
cilium-m66m7                       1/1     Running   1          20m
coredns-86f78bb79c-ttj2f           1/1     Running   0          45s
ubuntu@ubuntu1:~$ 
```