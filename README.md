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

Launch a few Ubuntu VM 

```bash
multipass launch -n ubuntu-k8smaster --cpus 6 --mem 8G --disk 30G
 ```

 Check that it all worked out

```console
root@b1s1-node1:~# multipass list
ubuntu-k8smaster        Running           10.57.89.33      Ubuntu 20.04 LTS
ubuntu-k8sworker-1      Running           10.57.89.165     Ubuntu 20.04 LTS
ubuntu-k8sworker-2      Running           10.57.89.205     Ubuntu 20.04 LTS```

## Microk8s Install 

```
multipass shell ubuntu-k8smaster
sudo snap install microk8s --classic
```
Result:
```console
root@b1s1-node1:~# multipass shell ubuntu-k8smaster
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-80-generic x86_64)
[...]
ubuntu@ubuntu-k8smaster:/etc$ sudo snap install microk8s --classic
microk8s (1.21/stable) v1.21.4 from Canonicalâœ“ installed
ubuntu@ubuntu-k8smaster:/etc$ 

```
Tune microk8s kubectl with k alias and autocompletion / Configure permissions 
```
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
echo "alias k='microk8s kubectl'" >>~/.bashrc
echo "complete -F __start_kubectl k" >>~/.bashrc
sudo bash -c "microk8s kubectl completion bash >/etc/bash_completion.d/kubectl"
exit
multipass shell ubuntu-k8smaster
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
```console
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
```

## multi node cluster

Deploy microk8s kubernetees on a second node (worker) and the appropriate permission settings.
Create several worker nodes in multipass on top of the master.
```console
root@b1s1-node1:~# multipass list
Name                    State             IPv4             Image
ubuntu-k8smaster        Running           10.57.89.33      Ubuntu 20.04 LTS
ubuntu-k8sworker-1      Running           10.57.89.165     Ubuntu 20.04 LTS
ubuntu-k8sworker-2      Running           10.57.89.205     Ubuntu 20.04 LTS
root@b1s1-node1:~# multipass shell ubuntu-k8sworker-1
[...]
ubuntu@ubuntu-k8sworker-1:~$ sudo snap install microk8s --classic
microk8s (1.21/stable) v1.21.4 from Canonicalâœ“ installed
[...]
```
And install microk8s on each worker node
```
multipass shell ubuntu-k8sworker-1
sudo snap install microk8s --classic
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
exit
multipass shell ubuntu-k8sworker-1
sudo snap install microk8s --classic
```

On master node, move back to calico (if cilium is running) and get the commands for copy-pasting to worker node
``` 
microk8s disable cilium
microk8s add-node
```
Example
```
#### On master node #####
ubuntu@ubuntu-k8smaster:/etc$ microk8s disable cilium
Disabling Cilium
[...]
ubuntu@ubuntu-k8smaster:~$ microk8s add-node
From the node you wish to join to this cluster, run the following:
microk8s join 10.57.89.33:25000/78dfefed0dcd09e2ad16a9ecc24ea33e/2c6583b08291

If the node you are adding is not reachable through the default interface you can use one of the following:
 microk8s join 10.57.89.33:25000/78dfefed0dcd09e2ad16a9ecc24ea33e/2c6583b08291
 microk8s join 10.0.0.199:25000/78dfefed0dcd09e2ad16a9ecc24ea33e/2c6583b08291
ubuntu@ubuntu-k8smaster:~$

#### On worker node #####

ubuntu@ubuntu-k8sworker-1:~$ microk8s join 10.57.89.33:25000/5acea0af7895b2ab3f2b10dd89af284a/2c6583b08291
Contacting cluster at 10.57.89.33
Waiting for this node to finish joining the cluster. ..  
ubuntu@ubuntu-k8sworker-1:~$
```

Wait for some time for things to complete... You should see the worker node added to the cluster.

```
ubuntu@ubuntu-k8smaster:~$ k get nodes
NAME                 STATUS   ROLES    AGE     VERSION
ubuntu-k8sworker-1   Ready    <none>   2m51s   v1.21.4-3+e5758f73ed2a04
ubuntu-k8smaster     Ready    <none>   103m    v1.21.4-3+e5758f73ed2a04
ubuntu@ubuntu-k8smaster:~$ 
```
Repeat the same on a second worker node. You should be fine
If you want to run cilium you can re-enable it after all nodes have joined. 
```
ubuntu@ubuntu-k8smaster:~$ microk8s enable cilium
Addon helm3 is already enabled.
[...]
ubuntu@ubuntu-k8smaster:/etc$ k get nodes -o wide
NAME                 STATUS   ROLES    AGE     VERSION                    INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
ubuntu-k8sworker-1   Ready    <none>   50m     v1.21.4-3+e5758f73ed2a04   10.57.89.165   <none>        Ubuntu 20.04.2 LTS   5.4.0-80-generic   containerd://1.4.4
ubuntu-k8smaster     Ready    <none>   151m    v1.21.4-3+e5758f73ed2a04   10.57.89.33    <none>        Ubuntu 20.04.2 LTS   5.4.0-80-generic   containerd://1.4.4
ubuntu-k8sworker-2   Ready    <none>   4m47s   v1.21.4-3+e5758f73ed2a04   10.57.89.205   <none>        Ubuntu 20.04.2 LTS   5.4.0-80-generic   containerd://1.4.4
ubuntu@ubuntu-k8smaster:/etc$ 
```
k apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml

## Appendix / Lesson learnt

### Useful adds-on
- Add core-dns
```console
ubuntu@ubuntu-k8smaster:~$ microk8s enable dns
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

ubuntu@ubuntu-k8smaster:~$ k get pods -n kube-system
NAME                               READY   STATUS    RESTARTS   AGE
cilium-operator-774f85cdd8-t2mcj   1/1     Running   1          20m
cilium-m66m7                       1/1     Running   1          20m
coredns-86f78bb79c-ttj2f           1/1     Running   0          45s
```

## snap/microk8s useful file location

Relevant configuration files are located there. CNI configs and binaries for example !

```
root@ubuntu-k8smaster:/opt/containerd# cd /var/snap/microk8s/current/args
root@ubuntu-k8smaster:/var/snap/microk8s/current/args# ls
cluster-agent  containerd      containerd-template.toml  ctr   flannel-network-mgr-config  flanneld        kube-controller-manager  kube-scheduler  kubectl-env
cni-network    containerd-env  containerd.toml           etcd  flannel-template.conflist   kube-apiserver  kube-proxy               kubectl         kubelet

root@ubuntu-k8sworker-2:~# cd /var/snap/microk8s/current/opt/cni/bin/
root@ubuntu-k8sworker-2:/var/snap/microk8s/current/opt/cni/bin# ls
bandwidth  calico       cilium-cni  firewall  flanneld     host-local  loopback  portmap  sbr     tuning
bridge     calico-ipam  dhcp        flannel   host-device  ipvlan      macvlan   ptp      static  vlan
root@ubuntu-k8sworker-2:/var/snap/microk8s/current/opt/cni/bin# ./cilium-cni
Cilium CNI plugin 1.8.12 b00a133 2021-09-01T12:51:30-07:00 go version go1.14.15 linux/amd64
root@ubuntu-k8sworker-2:/var/snap/microk8s/current/opt/cni/bin# 
```


### Yet Another CrashLoopBackoff Troubleshooting 

```
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
```
In microk8s, service configurations are located in /var/snap/microk8s/current/args.

For example, we can check info from containerd, which has also explicit references to cni config locations.

```console
root@ubuntu1:/var/snap/microk8s/current/args# cat containerd
--config ${SNAP_DATA}/args/containerd.toml
--root ${SNAP_COMMON}/var/lib/containerd
--state ${SNAP_COMMON}/run/containerd
--address ${SNAP_COMMON}/run/containerd.sock
root@ubuntu1

root@ubuntu1:/var/snap/microk8s/current/args# cat containerd.toml 
# Use config version 2 to enable new configuration fields.
version = 2
oom_score = 0

[grpc]
  uid = 0
  gid = 0
  max_recv_message_size = 16777216
  max_send_message_size = 16777216

[debug]
  address = ""
  uid = 0
  gid = 0

[metrics]
  address = "127.0.0.1:1338"
  grpc_histogram = false

[cgroup]
  path = ""


# The 'plugins."io.containerd.grpc.v1.cri"' table contains all of the server options.
[plugins."io.containerd.grpc.v1.cri"]

  stream_server_address = "127.0.0.1"
  stream_server_port = "0"
  enable_selinux = false
  sandbox_image = "k8s.gcr.io/pause:3.1"
  stats_collect_period = 10
  enable_tls_streaming = false
  max_container_log_line_size = 16384

  # 'plugins."io.containerd.grpc.v1.cri".containerd' contains config related to containerd
  [plugins."io.containerd.grpc.v1.cri".containerd]

    # snapshotter is the snapshotter used by containerd.
    snapshotter = "overlayfs"

    # no_pivot disables pivot-root (linux only), required when running a container in a RamDisk with runc.
    # This only works for runtime type "io.containerd.runtime.v1.linux".
    no_pivot = false

    # default_runtime_name is the default runtime name to use.
    default_runtime_name = "runc"

    # 'plugins."io.containerd.grpc.v1.cri".containerd.runtimes' is a map from CRI RuntimeHandler strings, which specify types
    # of runtime configurations, to the matching configurations.
    # In this example, 'runc' is the RuntimeHandler string to match.
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
      # runtime_type is the runtime type to use in containerd e.g. io.containerd.runtime.v1.linux
      runtime_type = "io.containerd.runc.v1"

    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia-container-runtime]
      # runtime_type is the runtime type to use in containerd e.g. io.containerd.runtime.v1.linux
      runtime_type = "io.containerd.runc.v1"

      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia-container-runtime.options]
        BinaryName = "nvidia-container-runtime"

  # 'plugins."io.containerd.grpc.v1.cri".cni' contains config related to cni
  [plugins."io.containerd.grpc.v1.cri".cni]
    # bin_dir is the directory in which the binaries for the plugin is kept.
    bin_dir = "/var/snap/microk8s/2060/opt/cni/bin"

    # conf_dir is the directory in which the admin places a CNI conf.
    conf_dir = "/var/snap/microk8s/2060/args/cni-network"

  # 'plugins."io.containerd.grpc.v1.cri".registry' contains config related to the registry
  [plugins."io.containerd.grpc.v1.cri".registry]

    # 'plugins."io.containerd.grpc.v1.cri".registry.mirrors' are namespace to mirror mapping for all namespaces.
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
        endpoint = ["https://registry-1.docker.io", ]
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors."localhost:32000"]
        endpoint = ["http://localhost:32000"]
```

Here it is possible to get a higher  log level for containerd, since it seems that something fails here.

```console
root@ubuntu1:/var/snap/microk8s/current/args# vi containerd
--config ${SNAP_DATA}/args/containerd.toml
--root ${SNAP_COMMON}/var/lib/containerd
--state ${SNAP_COMMON}/run/containerd
--address ${SNAP_COMMON}/run/containerd.sock
--log-level debug
```

Ultimately, after new journalctl, log show no error at all... The answer was very basic: container just stops normally. To fix that, just add "command sleep infinity" to the container spec.

```
ubuntu@ubuntu1:/var/snap/microk8s/current/args$ k edit deployment
[...]
    spec:
      containers:
      - image: alpine
        imagePullPolicy: Always
        name: alpine
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        command: [ "sleep" ]
        args: [ "infinity" ]
      dnsPolicy: ClusterFirst

```

And Voila...

```console
ubuntu@ubuntu1:/var/snap/microk8s/current/args$ k get pods
NAME                      READY   STATUS    RESTARTS   AGE
alpine-7dd86575d6-fk8q4   1/1     Running   0          4m24s
alpine-7dd86575d6-m6cbg   1/1     Running   0          4m19s
ubuntu@ubuntu1:/var/snap/microk8s/current/args$ 
```
## Another issue after moving from calico to cilium

After enabling cilium in the multi-node clusters via "microk8s enable cilium", things get broken. A deployment is stuck in "ContainerCreating". Obviously something kubelet can't create the networking interfaces.
```console
ubuntu@ubuntu-k8smaster:~$ k get pods
NAME                                READY   STATUS              RESTARTS   AGE
nginx-deployment-66b6c48dd5-8tznf   0/1     ContainerCreating   0          36m
nginx-deployment-66b6c48dd5-vzz99   0/1     ContainerCreating   0          36m
nginx-deployment-66b6c48dd5-2kzwn   1/1     Running             0          36m
ubuntu@ubuntu-k8smaster:~$
ubuntu@ubuntu-k8smaster:~$ k describe  pod nginx-deployment-66b6c48dd5-8tznf
Name:           nginx-deployment-66b6c48dd5-8tznf
Namespace:      default
[...]
  Warning  FailedCreatePodSandBox  2m17s (x152 over 35m)  kubelet            (combined from similar events): Failed to create pod sandbox: rpc error: code = Unknown desc = failed to setup network for sandbox "b8be46e6b79acf136141d7ceff0c38154b7843d6d81fb7cc894733b9dae013ab": error getting ClusterInformation: connection is unauthorized: Unauthorized
ubuntu@ubuntu-k8smaster:~$ 
```

The problem is reported by kubelet. After quick sanity check, we check the kubelet logs (btw kubelet is called kubelite in microk8s).

```console
ubuntu@ubuntu-k8smaster:/var/snap/microk8s/2407/args/cni-network$ k get nodes -o wide
NAME                 STATUS   ROLES    AGE     VERSION                    INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
ubuntu-k8sworker-2   Ready    <none>   4h54m   v1.21.4-3+e5758f73ed2a04   10.57.89.205   <none>        Ubuntu 20.04.2 LTS   5.4.0-80-generic   containerd://1.4.4
ubuntu-k8sworker-1   Ready    <none>   5h40m   v1.21.4-3+e5758f73ed2a04   10.57.89.165   <none>        Ubuntu 20.04.2 LTS   5.4.0-80-generic   containerd://1.4.4
ubuntu-k8smaster     Ready    <none>   7h20m   v1.21.4-3+e5758f73ed2a04   10.57.89.33    <none>        Ubuntu 20.04.2 LTS   5.4.0-80-generic   containerd://1.4.4
ubuntu@ubuntu-k8smaster:/var/snap/microk8s/2407/args/cni-network$ 

ubuntu@ubuntu-k8smaster:/var/snap/microk8s/2407/args/cni-network$ k get pods -n kube-system
NAME                               READY   STATUS    RESTARTS   AGE
cilium-operator-6997f6d645-8j9wm   1/1     Running   0          4h45m
cilium-bqnsc                       1/1     Running   0          4h45m
cilium-vhdkn                       1/1     Running   0          4h45m
cilium-c559d                       1/1     Running   1          4h45m
coredns-7f9c69c78c-zgmdg           0/1     Running   77         6h14m
ubuntu@ubuntu-k8smaster:/var/snap/microk8s/2407/args/cni-network$ 

root@ubuntu-k8sworker-1:/var/log# journalctl -f -u snap.microk8s.daemon-kubelite
-- Logs begin at Tue 2021-06-22 06:15:55 PDT. --
Sep 09 13:18:38 ubuntu-k8sworker-1 microk8s.daemon-kubelite[98983]: E0909 13:18:38.246991   98983 pod_workers.go:190] "Error syncing pod, skipping" err="failed to \"CreatePodSandbox\" for \"nginx-deployment-66b6c48dd5-8tznf_default(7f759422-7124-45a1-b5e3-64df840a6b74)\" with CreatePodSandboxError: \"Failed to create sandbox for pod \\\"nginx-deployment-66b6c48dd5-8tznf_default(7f759422-7124-45a1-b5e3-64df840a6b74)\\\": rpc error: code = Unknown desc = failed to setup network for sandbox \\\"3382c59c50a96f06dbf51260bf13b920e499eef8d29a5056bc22a97e20662754\\\": error getting ClusterInformation: connection is unauthorized: Unauthorized\"" pod="default/nginx-deployment-66b6c48dd5-8tznf" podUID=7f759422-7124-45a1-b5e3-64df840a6b74
Sep 09 13:18:48 ubuntu-k8sworker-1 microk8s.daemon-kubelite[98983]: E0909 13:18:48.603817   98983 authentication.go:63] "Unable to authenticate the request" err="[invalid bearer token, serviceaccounts \"calico-node\" not found]"
Sep 09 13:18:53 ubuntu-k8sworker-1 microk8s.daemon-kubelite[98983]: E0909 13:18:53.163053   98983 authentication.go:63] "Unable to authenticate the request" err="[invalid bearer token, serviceaccounts \"calico-node\" not found]"
Sep 09 13:18:53 ubuntu-k8sworker-1 microk8s.daemon-kubelite[98983]: E0909 13:18:53.236631   98983 authentication.go:63] "Unable to authenticate the request" err="[invalid bearer token, serviceaccounts \"calico-node\" not found]"
Sep 09 13:18:53 ubuntu-k8sworker-1 microk8s.daemon-kubelite[98983]: E0909 13:18:53.244914   98983 remote_runtime.go:116] "RunPodSandbox from runtime service failed" err="rpc error: code = Unknown desc = failed to setup network for sandbox \"a83bc7306739462809393ffc0ba5e349858a4ec2ed923130a420f94af1f36a70\": error getting ClusterInformation: connection is unauthorized: Unauthorized"
Sep 09 13:18:53 ubuntu-k8sworker-1 microk8s.daemon-kubelite[98983]: E0909 13:18:53.245007   98983 kuberuntime_sandbox.go:68] "Failed to create sandbox for pod" err="rpc error: code = Unknown desc = failed to setup network for sandbox \"a83bc7306739462809393ffc0ba5e349858a4ec2ed923130a420f94af1f36a70\": error getting ClusterInformation: connection is unauthorized: Unauthorized" pod="default/nginx-deployment-66b6c48dd5-8tznf"
Sep 09 13:18:53 ubuntu-k8sworker-1 microk8s.daemon-kubelite[98983]: E0909 13:18:53.245097   98983 kuberuntime_manager.go:790] "CreatePodSandbox for pod failed" err="rpc error: code = Unknown desc = failed to setup network for sandbox \"a83bc7306739462809393ffc0ba5e349858a4ec2ed923130a420f94af1f36a70\": error getting ClusterInformation: connection is unauthorized: Unauthorized" pod="default/nginx-deployment-66b6c48dd5-8tznf"
Sep 09 13:18:53 ubuntu-k8sworker-1 microk8s.daemon-kubelite[98983]: E0909 13:18:53.245195   98983 pod_workers.go:190] "Error syncing pod, skipping" err="failed to \"CreatePodSandbox\" for \"nginx-deployment-66b6c48dd5-8tznf_default(7f759422-7124-45a1-b5e3-64df840a6b74)\" with CreatePodSandboxError: \"Failed to create sandbox for pod \\\"nginx-deployment-66b6c48dd5-8tznf_default(7f759422-7124-45a1-b5e3-64df840a6b74)\\\": rpc error: code = Unknown desc = failed to setup network for sandbox \\\"a83bc7306739462809393ffc0ba5e349858a4ec2ed923130a420f94af1f36a70\\\": error getting ClusterInformation: connection is unauthorized: Unauthorized\"" pod="default/nginx-deployment-66b6c48dd5-8tznf" podUID=7f759422-7124-45a1-b5e3-64df840a6b74
Sep 09 13:18:58 ubuntu-k8sworker-1 microk8s.daemon-kubelite[98983]: E0909 13:18:58.096328   98983 remote_runtime.go:144] "StopPodSandbox from runtime service failed" err="rpc error: code = Unknown desc = failed to destroy network for sandbox \"2a44ad549148a03f2a94647191ff1c80332c61f9ee8c6110d09600fff2ef1a58\": error getting ClusterInformation: connection is unauthorized: Unauthorized" podSandboxID="2a44ad549148a03f2a94647191ff1c80332c61f9ee8c6110d09600fff2ef1a58"
Sep 09 13:18:58 ubuntu-k8sworker-1 microk8s.daemon-kubelite[98983]: E0909 13:18:58.096416   98983 kuberuntime_gc.go:176] "Failed to stop sandbox before removing" err="rpc error: code = Unknown desc = failed to destroy network for sandbox \"2a44ad549148a03f2a94647191ff1c80332c61f9ee8c6110d09600fff2ef1a58\": error getting ClusterInformation: connection is unauthorized: Unauthorized" sandboxID="2a44ad549148a03f2a94647191ff1c80332c61f9ee8c6110d09600fff2ef1a58"
Sep 09 13:19:00 ubuntu-k8sworker-1 microk8s.daemon-kubelite[98983]: E0909 13:19:00.605810   98983 authentication.go:63] "Unable to authenticate the request" err="[invalid bearer token, serviceaccounts \"calico-node\" not found]"
Sep 09 13:19:06 ubuntu-k8sworker-1 microk8s.daemon-kubelite[98983]: E0909 13:19:06.259402   98983 remote_runtime.go:116] "RunPodSandbox from runtime service failed" err="rpc error: code = Unknown desc = failed to setup network for sandbox \"e3b7881f804e8e853d2ba862b755545eb85c635191de5a755f6cac2bc2fe88cd\": error getting ClusterInformation: connection is unauthorized: Unauthorized"
Sep 09 13:19:06 ubuntu-k8sworker-1 microk8s.daemon-kubelite[98983]: E0909 13:19:06.259501   98983 kuberuntime_sandbox.go:68] "Failed to create sandbox for pod" err="rpc error: code = Unknown desc = failed to setup network for sandbox \"e3b7881f804e8e853d2ba862b755545eb85c635191de5a755f6cac2bc2fe88cd\": error getting ClusterInformation: connection is unauthorized: Unauthorized" pod="default/nginx-deployment-66b6c48dd5-8tznf"
Sep 09 13:19:06 ubuntu-k8sworker-1 microk8s.daemon-kubelite[98983]: E0909 13:19:06.259533   98983 kuberuntime_manager.go:790] "CreatePodSandbox for pod failed" err="rpc error: code = Unknown desc = failed to setup network for sandbox \"e3b7881f804e8e853d2ba862b755545eb85c635191de5a755f6cac2bc2fe88cd\": error getting ClusterInformation: connection is unauthorized: Unauthorized" pod="default/nginx-deployment-66b6c48dd5-8tznf"
Sep 09 13:19:06 ubuntu-k8sworker-1 microk8s.daemon-kubelite[98983]: E0909 13:19:06.259642   98983 pod_workers.go:190] "Error syncing pod, skipping" err="failed to \"CreatePodSandbox\" for \"nginx-deployment-66b6c48dd5-8tznf_default(7f759422-7124-45a1-b5e3-64df840a6b74)\" with CreatePodSandboxError: \"Failed to create sandbox for pod \\\"nginx-deployment-66b6c48dd5-8tznf_default(7f759422-7124-45a1-b5e3-64df840a6b74)\\\": rpc error: code = Unknown desc = failed to setup network for sandbox \\\"e3b7881f804e8e853d2ba862b755545eb85c635191de5a755f6cac2bc2fe88cd\\\": error getting ClusterInformation: connection is unauthorized: Unauthorized\"" pod="default/nginx-deployment-66b6c48dd5-8tznf" podUID=7f759422-7124-45a1-b5e3-64df840a6b74
Sep 09 13:19:12 ubuntu-k8sworker-1 microk8s.daemon-kubelite[98983]: I0909 13:19:12.129949   98983 client.go:360] parsed scheme: "passthrough"
Sep 09 13:19:12 ubuntu-k8sworker-1 microk8s.daemon-kubelite[98983]: I0909 13:19:12.130042   98983 passthrough.go:48] ccResolverWrapper: sending update to cc: {[{unix:///var/snap/microk8s/2407/var/kubernetes/backend//kine.sock  <nil> 0 <nil>}] <nil> <nil>}
Sep 09 13:19:12 ubuntu-k8sworker-1 microk8s.daemon-kubelite[98983]: I0909 13:19:12.130065   98983 clientconn.go:948] ClientConn switching balancer to "pick_first"
Sep 09 13:19:17 ubuntu-k8sworker-1 microk8s.daemon-kubelite[98983]: E0909 13:19:17.253496   98983 remote_runtime.go:116] "RunPodSandbox from runtime service failed" err="rpc error: code = Unknown desc = failed to setup network for sandbox \"ecf42f7d111cdf750bed3c8b831c8a261c818c251e87287d713de3a76fdfc7a9\": error getting ClusterInformation: connection is unauthorized: Unauthorized"
```
So we're having some issues there !  After some digging (complains with network and authentication), we see that cilium cni is not part of the kubelet cni directory. Again in microk8S things are all under this snap directory (just issue ps aux | grep kube and check cmdline options to find it ).

Start with refreshing certificates.
```
ubuntu@ubuntu-k8sworker-2:~$ sudo microk8s refresh-certs
Taking a backup of the current certificates under /var/snap/microk8s/2407/var/log/ca-backup/
Creating new certificates
Can't load /root/.rnd into RNG
140462069982656:error:2406F079:random number generator:RAND_load_file:Cannot open file:../crypto/rand/randfile.c:88:Filename=/root/.rnd
Can't load /root/.rnd into RNG
140367804642752:error:2406F079:random number generator:RAND_load_file:Cannot open file:../crypto/rand/randfile.c:88:Filename=/root/.rnd
Signature ok
subject=C = GB, ST = Canonical, L = Canonical, O = Canonical, OU = Canonical, CN = 127.0.0.1
Getting CA Private Key
```
Next fix the cni config.
```
### options for kubelet ###
ubuntu@ubuntu-k8sworker-2:~$ cat /var/snap/microk8s/2407/args/kubelet
--kubeconfig=${SNAP_DATA}/credentials/kubelet.config
--cert-dir=${SNAP_DATA}/certs
--client-ca-file=${SNAP_DATA}/certs/ca.crt
--anonymous-auth=false
--network-plugin=cni
--root-dir=${SNAP_COMMON}/var/lib/kubelet
--fail-swap-on=false
--cni-conf-dir=${SNAP_DATA}/args/cni-network/
--cni-bin-dir=${SNAP_DATA}/opt/cni/bin/
--feature-gates=DevicePlugins=true
--eviction-hard="memory.available<100Mi,nodefs.available<1Gi,imagefs.available<1Gi"
--container-runtime=remote
--container-runtime-endpoint=${SNAP_COMMON}/run/containerd.sock
--containerd=${SNAP_COMMON}/run/containerd.sock
--node-labels="microk8s.io/cluster=true"
--authentication-token-webhook=true
--cluster-domain=cluster.local
--cluster-dns=10.152.183.10
### check the cni-network folder (cni config)
ubuntu@ubuntu-k8sworker-2:~$ ls /var/snap/microk8s/2407/args/cni-network/
10-calico.conflist  calico-kubeconfig  cni.yaml  cni.yaml.backup
ubuntu@ubuntu-k8sworker-2:~$
```
For some reason, cilium is missing (and calico was obviously not cleaned up properly). So pick the config from the master (which work).
```console
### run on master ###
ubuntu@ubuntu-k8smaster:/var/snap/microk8s/2407/args/cni-network$ ls
05-cilium-cni.conf  10-calico.conflist  calico-kubeconfig  cni.yaml.backup  cni.yaml.disabled
ubuntu@ubuntu-k8smaster:/var/snap/microk8s/2407/args/cni-network$ cat 05-cilium-cni.conf 
{
    "cniVersion": "0.3.1",
    "name": "cilium",
    "type": "cilium-cni",
    "enable-debug": true
}
### copy to worker ###

ubuntu@ubuntu-k8sworker-2:/var/snap/microk8s/2407/args/cni-network$ sudo -i
root@ubuntu-k8sworker-2:~# cd /var/snap/microk8s/2407/args/cni-network
root@ubuntu-k8sworker-2:/var/snap/microk8s/2407/args/cni-network#  cat << EOF >> 05-cilium-cni.conf
> {
>     "cniVersion": "0.3.1",
>     "name": "cilium",
>     "type": "cilium-cni",
>     "enable-debug": true
> }
> EOF
```
After these steps on all workers, the pods can be deployed on all workers.
```console
ubuntu@ubuntu-k8smaster:/var/snap/microk8s/2407/args/cni-network$ k get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE     IP           NODE                 NOMINATED NODE   READINESS GATES
nginx-deployment-66b6c48dd5-2kzwn   1/1     Running   0          5h34m   10.0.0.107   ubuntu-k8smaster     <none>           <none>
nginx-deployment-66b6c48dd5-8tznf   1/1     Running   1          5h34m   10.0.1.199   ubuntu-k8sworker-1   <none>           <none>
nginx-deployment-66b6c48dd5-vzz99   1/1     Running   0          5h34m   10.0.2.174   ubuntu-k8sworker-2   <none>           <none>
ubuntu@ubuntu-k8smaster:/var/snap/microk8s/2407/args/cni-network$ 
```

## Upgrade
Tried but did not work

```
ubuntu@ubuntu-k8smaster:/etc$ k version
Client Version: version.Info{Major:"1", Minor:"19+", GitVersion:"v1.19.13-34+939585d5fb6fa7", GitCommit:"939585d5fb6fa724a79db2b43cff42342706a146", GitTreeState:"clean", BuildDate:"2021-07-16T20:58:41Z", GoVersion:"go1.15.14", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"19+", GitVersion:"v1.19.13-34+939585d5fb6fa7", GitCommit:"939585d5fb6fa724a79db2b43cff42342706a146", GitTreeState:"clean", BuildDate:"2021-07-16T20:59:39Z", GoVersion:"go1.15.14", Compiler:"gc", Platform:"linux/amd64"}

ubuntu@ubuntu-k8smaster:/etc$ snap install microk8s --classic --channel=1.22/stable
error: access denied (try with sudo)
ubuntu@ubuntu-k8smaster:/etc$ sudo snap refresh microk8s --channel=1.21/stable
Fetch and check assertions for snap "microk8s" (2407)                             ```                                                                                                     \

## Interesting links and reading

https://logz.io/blog/a-practical-guide-to-kubernetes-logging/

https://tharangarajapaksha.medium.com/start-k8s-with-microk8s-85b67738b557
