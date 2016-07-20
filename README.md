# rktnetes-workshop

This repo contains a Vagrantfile and necessary assets to help you launch your [Kubernetes](https://github.com/kubernetes/kubernetes) cluster with [rkt](https://github.com/coreos/rkt) as a runtime.

## Workshop

The workshop slides can be viewed at http://go-talks.appspot.com/github.com/coreos/rktnetes-workshop/workshop.slide.

## Vagrant up your rktnetes cluster.

```shell
$ cd vagrant
$ vagrant up
[...]

$ vagrant ssh

[vagrant@localhost ~]$ kubectl describe node 127.0.0.1
Name:			127.0.0.1
Labels:			beta.kubernetes.io/arch=amd64
			beta.kubernetes.io/os=linux
			kubernetes.io/hostname=127.0.0.1
Taints:			<none>
CreationTimestamp:	Mon, 27 Jun 2016 21:23:07 +0000
Phase:			
Conditions:
  Type			Status	LastHeartbeatTime			LastTransitionTime			Reason				Message
  ----			------	-----------------			------------------			------				-------
  OutOfDisk 		False 	Mon, 27 Jun 2016 21:25:10 +0000 	Mon, 27 Jun 2016 21:23:07 +0000 	KubeletHasSufficientDisk 	kubelet has sufficient disk space available
  MemoryPressure 	False 	Mon, 27 Jun 2016 21:25:10 +0000 	Mon, 27 Jun 2016 21:23:07 +0000 	KubeletHasSufficientMemory 	kubelet has sufficient memory available
  Ready 		True 	Mon, 27 Jun 2016 21:25:10 +0000 	Mon, 27 Jun 2016 21:23:07 +0000 	KubeletReady 			kubelet is posting ready status
Addresses:		127.0.0.1,127.0.0.1
Capacity:
 alpha.kubernetes.io/nvidia-gpu:	0
 cpu:					2
 memory:				2048712Ki
 pods:					110
Allocatable:
 alpha.kubernetes.io/nvidia-gpu:	0
 cpu:					2
 memory:				2048712Ki
 pods:					110
System Info:
 Machine ID:			508619e7e68e446c84d1fcdf7e0dc577
 System UUID:			B488DBD6-3470-4E69-BF62-DE81AD069083
 Boot ID:			1ea277cf-87e2-4383-bbea-305339975de9
 Kernel Version:		4.5.5-300.fc24.x86_64
 OS Image:			Fedora 24 (Cloud Edition)
 Operating System:		linux
 Architecture:			amd64
 Container Runtime Version:	rkt://1.9.1
 Kubelet Version:		v1.3.0-beta.2
 Kube-Proxy Version:		v1.3.0-beta.2
ExternalID:			127.0.0.1
Non-terminated Pods:		(0 in total)
  Namespace			Name		CPU Requests	CPU Limits	Memory Requests	Memory Limits
  ---------			----		------------	----------	---------------	-------------
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted. More info: http://releases.k8s.io/HEAD/docs/user-guide/compute-resources.md)
  CPU Requests	CPU Limits	Memory Requests	Memory Limits
  ------------	----------	---------------	-------------
  0 (0%)	0 (0%)		0 (0%)		0 (0%)
Events:
  FirstSeen	LastSeen	Count	From			SubobjectPath	Type		Reason			Message
  ---------	--------	-----	----			-------------	--------	------			-------
  2m		2m		1	{kube-proxy 127.0.0.1}			Normal		Starting		Starting kube-proxy.
  2m		2m		1	{kubelet 127.0.0.1}			Normal		Starting		Starting kubelet.
  2m		2m		7	{kubelet 127.0.0.1}			Normal		NodeHasSufficientDisk	Node 127.0.0.1 status is now: NodeHasSufficientDisk
  2m		2m		7	{kubelet 127.0.0.1}			Normal		NodeHasSufficientMemory	Node 127.0.0.1 status is now: NodeHasSufficientMemory
  2m		2m		1	{controllermanager }			Normal		RegisteredNode		Node 127.0.0.1 event: Registered Node 127.0.0.1 in NodeController

```

Now we can start playing with the rktnetes cluster!

```shell
[vagrant@localhost ~]$ kubectl run redis --image=redis
deployment "redis" created

[vagrant@localhost ~]$ kubectl get pods
NAME                     READY     STATUS    RESTARTS   AGE
redis-4282552436-bwue8   1/1       Running   0          13s

[vagrant@localhost ~]$ rkt list
UUID		APP		IMAGE NAME					STATE	CREATED		STARTED		NETWORKS
2a827036	hyperkube	quay.io/coreos/hyperkube:v1.3.0-beta.2_coreos.0	running	14 minutes ago	14 minutes ago	
72210e51	etcd		quay.io/coreos/etcd:v2.3.7			running	14 minutes ago	14 minutes ago	
928296f0	hyperkube	quay.io/coreos/hyperkube:v1.3.0-beta.2_coreos.0	running	14 minutes ago	14 minutes ago	
97c2bef9	hyperkube	quay.io/coreos/hyperkube:v1.3.0-beta.2_coreos.0	running	14 minutes ago	14 minutes ago	
edee7641	redis		registry-1.docker.io/library/redis:latest	running	1 minute ago	1 minute ago	rkt.kubernetes.io:ip4=10.1.0.2, default-restricted:ip4=172.16.28.2
```

Add the Kubernetes Dashboard:

```shell
[vagrant@localhost ~]$ kubectl create -f https://rawgit.com/kubernetes/dashboard/master/src/deploy/kubernetes-dashboard.yaml
```

Add an Ingress controller based on [traefik](https://traefik.io/):

```shell
[vagrant@localhost ~]$ kubectl create -f /vagrant/traefik.yaml
```

Modify your local `/etc/hosts`, replace the entry with the Vagrant IP address:
```
[vagrant@localhost ~]$ ip addr show dev eth1
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:90:3f:9a brd ff:ff:ff:ff:ff:ff
    inet 172.28.128.126/24 brd 172.28.128.255 scope global dynamic eth1
       valid_lft 3221sec preferred_lft 3221sec
    inet6 fe80::5054:ff:fe90:3f9a/64 scope link 
       valid_lft forever preferred_lft forever

[vagrant@localhost ~]$ exit

$ vi /etc/hosts
...
172.28.128.126    traefik.rktnetes
```

Open http://traefik.rktnetes.
