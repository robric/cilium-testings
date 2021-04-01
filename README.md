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

## VM installation

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

## Microk8s Install 

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

Troubleshooting CrashLoopBackoff. 
First there is not much info in describe... Unfortunately.

```console
Events:
  Type     Reason     Age                   From               Message
  ----     ------     ----                  ----               -------
  Normal   Scheduled  12m                   default-scheduler  Successfully assigned default/alpine-6b967c77f7-nbdtd to ubuntu1
  Normal   Pulled     12m                   kubelet            Successfully pulled image "alpine" in 2.716907009s
  Normal   Pulled     12m                   kubelet            Successfully pulled image "alpine" in 1.558567615s
  Normal   Pulled     12m                   kubelet            Successfully pulled image "alpine" in 1.663127517s
  Normal   Pulled     11m                   kubelet            Successfully pulled image "alpine" in 1.243626204s
  Normal   Created    11m (x4 over 12m)     kubelet            Created container alpine
  Normal   Started    11m (x4 over 12m)     kubelet            Started container alpine
  Normal   Pulling    10m (x5 over 12m)     kubelet            Pulling image "alpine"
  Warning  BackOff    2m40s (x46 over 12m)  kubelet            Back-off restarting failed container
ubuntu@ubuntu1:~$
```
Nor there is in log (kubectl log or /var/logs/pods/ID)
Let's have a look a runtime level (containerd)
```console
ubuntu@ubuntu1:~$ microk8s ctr container list
CONTAINER                                                           IMAGE                                                                                               RUNTIME                  
0bf7cf0f5fcb2033d86c1003a8f2b14570d5bba16d03647f2b0066a5fe494ae9    k8s.gcr.io/pause:3.1                                                                                io.containerd.runc.v1    
12fb6c5332acdd096ee3bdc6f97129f044c58a8e64248acf5ad8b7b5c8350e83    k8s.gcr.io/pause:3.1                                                                                io.containerd.runc.v1    
14ba7077eae160aba180967bb8dc4a20688ec28e6bb963dbd937b5e40d50497c    docker.io/library/alpine@sha256:ec14c7992a97fc11425907e908340c6c3d6ff602f5f13d899e6b7027c9b4133a    io.containerd.runc.v1    
2ebeede3e2ab091b7d4caaf1380acb7f89240ce38eb2c3aa7d0a6b64f8c9df2c    k8s.gcr.io/pause:3.1                                                                                io.containerd.runc.v1    
3655f0408c9d6c2365ccb6d30512af2cba1b4a4742be559b916e0d899dc0b020    docker.io/coredns/coredns:1.6.6                                                                     io.containerd.runc.v1    
3bdb9471be457e06cb5879125830f3db819648fdfea7b0ec1baac2e90eac7e2a    k8s.gcr.io/pause:3.1                                                                                io.containerd.runc.v1    
42fde1f3214ee5dcaf9206a365fff1aee44aa6d96773d2176d684d6151fc0141    k8s.gcr.io/pause:3.1                                                                                io.containerd.runc.v1    
5312546f50b8aede1a13fda50fce533e64a9c7deb21a95234d99ccd638993de9    k8s.gcr.io/pause:3.1                                                                                io.containerd.runc.v1    
79b05fe4fcbdfa8845479538c0b2792842d8113131072c395c2ccd4ab7c4bbdd    docker.io/cilium/cilium:v1.6.12                                                                     io.containerd.runc.v1    
7c7dea55854c439139ac12e2e3f7ac85d286d726904e11f3b6ba0ddaeb56792e    docker.io/cilium/operator:v1.6.12                                                                   io.containerd.runc.v1    
af74630d4b85bc81f5ce2dcf2ef1cfd663a6ecbdd4737f09b9a67279c00d367f    docker.io/cilium/operator:v1.6.12                                                                   io.containerd.runc.v1    
c3342dbde708715265fd738ec0973412b090b838c04e154f8b15ef160c63583d    docker.io/cilium/cilium:v1.6.12                                                                     io.containerd.runc.v1    
c73e80f1fc979bc6c178b6edc20b37c84debd4c4f871a032c4df3e6f60936952    k8s.gcr.io/pause:3.1                                                                                io.containerd.runc.v1    
de955a318d59a7c5e41415aec78ea67c69ca351caae37f137b69d7d73dc2c30a    docker.io/cilium/cilium:v1.6.12                                                                     io.containerd.runc.v1    
e4fd33bf557bcb8842135290e1d005b05bf717a8baad09f5ecc4364042141630    docker.io/library/alpine@sha256:ec14c7992a97fc11425907e908340c6c3d6ff602f5f13d899e6b7027c9b4133a    io.containerd.runc.v1    
ubuntu@ubuntu1:~$ 
```
No logs as well. I can get an event related. 
```console
ubuntu@ubuntu1:~$ microk8s ctr  events
2021-04-01 11:50:27.322765234 +0000 UTC k8s.io /images/update {"name":"docker.io/library/alpine:latest","labels":{"io.cri-containerd.image":"managed"}}
2021-04-01 11:50:27.584558232 +0000 UTC k8s.io /images/update {"name":"sha256:49f356fa4513676c5e22e3a8404aad6c7262cc7aaed15341458265320786c58c","labels":{"io.cri-containerd.image":"managed"}}
2021-04-01 11:50:27.634697172 +0000 UTC k8s.io /images/update {"name":"docker.io/library/alpine:latest","labels":{"io.cri-containerd.image":"managed"}}
2021-04-01 11:50:27.667932215 +0000 UTC k8s.io /images/update {"name":"docker.io/library/alpine@sha256:ec14c7992a97fc11425907e908340c6c3d6ff602f5f13d899e6b7027c9b4133a","labels":{"io.cri-containerd.image":"managed"}}
2021-04-01 11:50:27.793033932 +0000 UTC k8s.io /snapshot/prepare {"key":"3cfb678e10f8481394a688d67aa2c9429f83c062cbb7ee0fb6d2e2a352a86929","parent":"sha256:8ea3b23f387bedc5e3cee574742d748941443c328a75f511eb37b0d8b6164130"}
2021-04-01 11:50:28.426144194 +0000 UTC k8s.io /containers/create {"id":"3cfb678e10f8481394a688d67aa2c9429f83c062cbb7ee0fb6d2e2a352a86929","image":"docker.io/library/alpine@sha256:ec14c7992a97fc11425907e908340c6c3d6ff602f5f13d899e6b7027c9b4133a","runtime":{"name":"io.containerd.runc.v1"}}
2021-04-01 11:50:28.76338842 +0000 UTC k8s.io /tasks/create {"container_id":"3cfb678e10f8481394a688d67aa2c9429f83c062cbb7ee0fb6d2e2a352a86929","bundle":"/var/snap/microk8s/common/run/containerd/io.containerd.runtime.v2.task/k8s.io/3cfb678e10f8481394a688d67aa2c9429f83c062cbb7ee0fb6d2e2a352a86929","rootfs":[{"type":"overlay","source":"overlay","options":["workdir=/var/snap/microk8s/common/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/132/work","upperdir=/var/snap/microk8s/common/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/132/fs","lowerdir=/var/snap/microk8s/common/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/72/fs"]}],"io":{"stdout":"/var/snap/microk8s/common/run/containerd/io.containerd.grpc.v1.cri/containers/3cfb678e10f8481394a688d67aa2c9429f83c062cbb7ee0fb6d2e2a352a86929/io/885532755/3cfb678e10f8481394a688d67aa2c9429f83c062cbb7ee0fb6d2e2a352a86929-stdout","stderr":"/var/snap/microk8s/common/run/containerd/io.containerd.grpc.v1.cri/containers/3cfb678e10f8481394a688d67aa2c9429f83c062cbb7ee0fb6d2e2a352a86929/io/885532755/3cfb678e10f8481394a688d67aa2c9429f83c062cbb7ee0fb6d2e2a352a86929-stderr"},"pid":151350}
```

journalctl has a bit more info, but nothing self-explanatory

```
ubuntu@ubuntu1:~$ journalctl -f -u snap.microk8s.daemon-kubelet 

Apr 01 05:33:40 ubuntu1 microk8s.daemon-kubelet[3596]: E0401 05:33:40.153836    3596 manager.go:1123] Failed to create existing container: /kubepods/besteffort/podf850bf3d-f42d-4619-a0bc-93113fca674b/55ca1d0ae66aaa48f2045db270f416c471a37a03c490db43f38153a960b0af07: task 55ca1d0ae66aaa48f2045db270f416c471a37a03c490db43f38153a960b0af07 not found: not found
Apr 01 05:33:40 ubuntu1 microk8s.daemon-kubelet[3596]: I0401 05:33:40.573887    3596 topology_manager.go:221] [topologymanager] RemoveContainer - Container ID: 361281aac25652d21c627b18bc71ebd71415bad33dc1964d8d7f4c46dca9d68b
Apr 01 05:33:40 ubuntu1 microk8s.daemon-kubelet[3596]: E0401 05:33:40.574726    3596 pod_workers.go:191] Error syncing pod 77c32b49-e7a5-419b-a11e-b12fbcd00132 ("alpine-6b967c77f7-nbdtd_default(77c32b49-e7a5-419b-a11e-b12fbcd00132)"), skipping: failed to "StartContainer" for "alpine" with CrashLoopBackOff: "back-off 5m0s restarting failed container=alpine pod=alpine-6b967c77f7-nbdtd_default(77c32b49-e7a5-419b-a11e-b12fbcd00132)"
Apr 01 05:33:41 ubuntu1 microk8s.daemon-kubelet[3596]: E0401 05:33:41.660315    3596 manager.go:1123] Failed to create existing container: /kubepods/besteffort/podf850bf3d-f42d-4619-a0bc-93113fca674b/b2c1606ab54d4f36dbddc8a8a81e2fe3611fabc754dd861d26cb4d7a6a926e1b: task b2c1606ab54d4f36dbddc8a8a81e2fe3611fabc754dd861d26cb4d7a6a926e1b not found: not found
Apr 01 05:33:44 ubuntu1 microk8s.daemon-kubelet[3596]: I0401 05:33:44.573886    3596 topology_manager.go:221] [topologymanager] RemoveContainer - Container ID: 69a74b5053b145726c33591f7a3c4291ac7702ab4defeab6fcbe75fecad101ba
Apr 01 05:33:44 ubuntu1 microk8s.daemon-kubelet[3596]: E0401 05:33:44.574374    3596 pod_workers.go:191] Error syncing pod 913d375a-dd90-43c1-ba19-a197488a69e4 ("alpine-6b967c77f7-2l48n_default(913d375a-dd90-43c1-ba19-a197488a69e4)"), skipping: failed to "StartContainer" for "alpine" with CrashLoopBackOff: "back-off 5m0s restarting failed container=alpine pod=alpine-6b967c77f7-2l48n_default(913d375a-dd90-43c1-ba19-a197488a69e4)"