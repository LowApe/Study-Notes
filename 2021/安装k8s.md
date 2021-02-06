# 安装k8s

> 安装 kubernets 官方文档很繁琐，但是kubesphere 提供了更方便的 kubeKey 安装方式，但是因为图形化占用资源，但是kubesphere 一键安装真的太方便了，虽然在安装kubesphere 我的cpu 因为资源不足，总有几个 pod 因为资源起不来。但是单方面安装kubernete 安装步骤也很多，所有现在思路是 kubesphere + Rancher 

[在 Linux 上以 All-in-One 模式安装 KubeSphere](https://kubesphere.com.cn/docs/quick-start/all-in-one-on-linux/)安装 k8s

```shell
# 安装相关依赖
$ apt install socat
$ apt install conntrack
$ apt install ebtables
$ apt install ipset
# 安装 docker
# 下载 kubeKey
$ export KKZONE=cn
$ curl -sfL https://get-kk.kubesphere.io | VERSION=v1.0.1 sh -
# kk 安装,只安装 kubernetes
$ ./kk create cluster --with-kubernetes v1.17.9
+--------+------+------+---------+----------+-------+-------+-----------+--------+------------+-------------+------------------+--------------+
| name   | sudo | curl | openssl | ebtables | socat | ipset | conntrack | docker | nfs client | ceph client | glusterfs client | time         |
+--------+------+------+---------+----------+-------+-------+-----------+--------+------------+-------------+------------------+--------------+
| ubuntu | y    | y    | y       | y        | y     | y     | y         | y      |            |             |                  | UTC 03:48:54 |
+--------+------+------+---------+----------+-------+-------+-----------+--------+------------+-------------+------------------+--------------+

This is a simple check of your environment.
Before installation, you should ensure that your machines meet all requirements specified at
https://github.com/kubesphere/kubekey#requirements-and-recommendations

Continue this installation? [yes/no]: yes
INFO[03:48:57 UTC] Downloading Installation Files
INFO[03:48:57 UTC] Downloading kubeadm ...
INFO[03:48:58 UTC] Downloading kubelet ...
INFO[03:48:59 UTC] Downloading kubectl ...
INFO[03:49:00 UTC] Downloading helm ...
INFO[03:49:01 UTC] Downloading kubecni ...
INFO[03:49:01 UTC] Configurating operating system ...
[ubuntu 192.168.199.177] MSG:
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-arptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_local_reserved_ports = 30000-32767
INFO[03:49:08 UTC] Installing docker ...
INFO[03:49:14 UTC] Start to download images on all nodes
[ubuntu] Downloading image: kubesphere/etcd:v3.3.12
[ubuntu] Downloading image: kubesphere/pause:3.1
[ubuntu] Downloading image: kubesphere/kube-apiserver:v1.17.9
[ubuntu] Downloading image: kubesphere/kube-controller-manager:v1.17.9
[ubuntu] Downloading image: kubesphere/kube-scheduler:v1.17.9
[ubuntu] Downloading image: kubesphere/kube-proxy:v1.17.9
[ubuntu] Downloading image: coredns/coredns:1.6.9
[ubuntu] Downloading image: kubesphere/k8s-dns-node-cache:1.15.12
[ubuntu] Downloading image: calico/kube-controllers:v3.15.1
[ubuntu] Downloading image: calico/cni:v3.15.1
[ubuntu] Downloading image: calico/node:v3.15.1
[ubuntu] Downloading image: calico/pod2daemon-flexvol:v3.15.1
INFO[03:53:08 UTC] Generating etcd certs
INFO[03:53:10 UTC] Synchronizing etcd certs
INFO[03:53:10 UTC] Creating etcd service
INFO[03:53:18 UTC] Starting etcd cluster
[ubuntu 192.168.199.177] MSG:
Configuration file will be created
INFO[03:53:20 UTC] Refreshing etcd configuration
Waiting for etcd to start
INFO[03:53:28 UTC] Backup etcd data regularly
INFO[03:53:29 UTC] Get cluster status
[ubuntu 192.168.199.177] MSG:
Cluster will be created.
INFO[03:53:30 UTC] Installing kube binaries
Push /home/didiaoyuan/kubesphere/kubekey/v1.17.9/amd64/kubeadm to 192.168.199.177:/tmp/kubekey/kubeadm   Done
Push /home/didiaoyuan/kubesphere/kubekey/v1.17.9/amd64/kubelet to 192.168.199.177:/tmp/kubekey/kubelet   Done
Push /home/didiaoyuan/kubesphere/kubekey/v1.17.9/amd64/kubectl to 192.168.199.177:/tmp/kubekey/kubectl   Done
Push /home/didiaoyuan/kubesphere/kubekey/v1.17.9/amd64/helm to 192.168.199.177:/tmp/kubekey/helm   Done
Push /home/didiaoyuan/kubesphere/kubekey/v1.17.9/amd64/cni-plugins-linux-amd64-v0.8.6.tgz to 192.168.199.177:/tmp/kubekey/cni-plugins-linux-amd64-v0.8.6.tgz   Done
INFO[03:53:42 UTC] Initializing kubernetes cluster
[ubuntu 192.168.199.177] MSG:
W0205 03:53:43.638264    9824 defaults.go:186] The recommended value for "clusterDNS" in "KubeletConfiguration" is: [10.233.0.10]; the provided value is: [169.254.25.10]
W0205 03:53:43.638546    9824 validation.go:28] Cannot validate kube-proxy config - no validator is available
W0205 03:53:43.638666    9824 validation.go:28] Cannot validate kubelet config - no validator is available
[init] Using Kubernetes version: v1.17.9
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
	[WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.3. Latest validated version: 19.03
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [ubuntu kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local lb.kubesphere.local kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local localhost lb.kubesphere.local ubuntu ubuntu.cluster.local] and IPs [10.233.0.1 192.168.199.177 127.0.0.1 192.168.199.177 10.233.0.1]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] External etcd mode: Skipping etcd/ca certificate authority generation
[certs] External etcd mode: Skipping etcd/server certificate generation
[certs] External etcd mode: Skipping etcd/peer certificate generation
[certs] External etcd mode: Skipping etcd/healthcheck-client certificate generation
[certs] External etcd mode: Skipping apiserver-etcd-client certificate generation
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[controlplane] Adding extra host path mount "host-time" to "kube-controller-manager"
W0205 03:53:53.604785    9824 manifests.go:214] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[controlplane] Adding extra host path mount "host-time" to "kube-controller-manager"
W0205 03:53:53.618430    9824 manifests.go:214] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[controlplane] Adding extra host path mount "host-time" to "kube-controller-manager"
W0205 03:53:53.623086    9824 manifests.go:214] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[kubelet-check] Initial timeout of 40s passed.
[apiclient] All control plane components are healthy after 52.512599 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.17" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node ubuntu as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node ubuntu as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: 0yhksq.novtyt2ytitxyoxt
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join lb.kubesphere.local:6443 --token 0yhksq.novtyt2ytitxyoxt \
    --discovery-token-ca-cert-hash sha256:df93accf7f429297fd061c1aec76697bf99890bbb0cfbae4839679b57592f730 \
    --control-plane

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join lb.kubesphere.local:6443 --token 0yhksq.novtyt2ytitxyoxt \
    --discovery-token-ca-cert-hash sha256:df93accf7f429297fd061c1aec76697bf99890bbb0cfbae4839679b57592f730
[ubuntu 192.168.199.177] MSG:
node/ubuntu untainted
[ubuntu 192.168.199.177] MSG:
node/ubuntu labeled
[ubuntu 192.168.199.177] MSG:
service "kube-dns" deleted
[ubuntu 192.168.199.177] MSG:
service/coredns created
[ubuntu 192.168.199.177] MSG:
serviceaccount/nodelocaldns created
daemonset.apps/nodelocaldns created
[ubuntu 192.168.199.177] MSG:
configmap/nodelocaldns created
[ubuntu 192.168.199.177] MSG:
W0205 03:55:27.083852   11854 version.go:101] could not fetch a Kubernetes version from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.txt": Get https://storage.googleapis.com/kubernetes-release/release/stable-1.txt: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
W0205 03:55:27.084012   11854 version.go:102] falling back to the local client version: v1.17.9
W0205 03:55:27.084254   11854 validation.go:28] Cannot validate kube-proxy config - no validator is available
W0205 03:55:27.084287   11854 validation.go:28] Cannot validate kubelet config - no validator is available
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
d7939e13b3724178014a229f473427d3a3a551f5ee176137abe2797c42631ba8
[ubuntu 192.168.199.177] MSG:
secret/kubeadm-certs patched
[ubuntu 192.168.199.177] MSG:
secret/kubeadm-certs patched
[ubuntu 192.168.199.177] MSG:
secret/kubeadm-certs patched
[ubuntu 192.168.199.177] MSG:
W0205 03:55:27.916900   12205 validation.go:28] Cannot validate kube-proxy config - no validator is available
W0205 03:55:27.916999   12205 validation.go:28] Cannot validate kubelet config - no validator is available
kubeadm join lb.kubesphere.local:6443 --token 6pbguq.saqwj52dugddej8h     --discovery-token-ca-cert-hash sha256:df93accf7f429297fd061c1aec76697bf99890bbb0cfbae4839679b57592f730
[ubuntu 192.168.199.177] MSG:
NAME     STATUS     ROLES           AGE   VERSION   INTERNAL-IP       EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
ubuntu   NotReady   master,worker   47s   v1.17.9   192.168.199.177   <none>        Ubuntu 18.04.5 LTS   4.15.0-135-generic   docker://20.10.3
INFO[03:55:28 UTC] Deploying network plugin ...
[ubuntu 192.168.199.177] MSG:
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
INFO[03:55:32 UTC] Joining nodes to cluster
INFO[03:55:32 UTC] Congradulations! Installation is successful.
# 查看基本信息
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-59d85c5c84-d2zkm   1/1     Running   0          74s
kube-system   calico-node-lnrzx                          1/1     Running   0          74s
kube-system   coredns-74d59cc5c6-tbtpr                   1/1     Running   0          103s
kube-system   coredns-74d59cc5c6-wcz7n                   1/1     Running   0          103s
kube-system   kube-apiserver-ubuntu                      1/1     Running   0          97s
kube-system   kube-controller-manager-ubuntu             1/1     Running   0          97s
kube-system   kube-proxy-k8cq2                           1/1     Running   0          103s
kube-system   kube-scheduler-ubuntu                      1/1     Running   0          97s
kube-system   nodelocaldns-kvv6v                         1/1     Running   0          101s
```



![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1gnchfcurlpj31e213mqaf.jpg)

- [Ubuntu安装Docker](https://www.runoob.com/docker/ubuntu-docker-install.html)



# 安装  PVC 使用 StorgaeClass

```shell
# 安装 StorageClass
$ kubectl create -f - <<EOF
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
EOF

# 静态绑定 pve
$ kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pve
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
  storageClassName: local-storage
$ kubectl get sc
NAME            PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-storage   kubernetes.io/no-provisioner   Delete          WaitForFirstConsumer   false                  7s
$ kubectl get pvc
NAME        STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS    AGE
local-pve   Pending                                      local-storage   29s
```

> 也可以放到 yaml 文件

# 安装 KubeSphere

```shell
$ kubectl apply -f kubesphere-installer.yaml
customresourcedefinition.apiextensions.k8s.io/clusterconfigurations.installer.kubesphere.io created
namespace/kubesphere-system created
serviceaccount/ks-installer created
clusterrole.rbac.authorization.k8s.io/ks-installer created
clusterrolebinding.rbac.authorization.k8s.io/ks-installer created
deployment.apps/ks-installer created

$ kubectl apply -f cluster-configuration.yaml
clusterconfiguration.installer.kubesphere.io/ks-installer created
```



# 安装Rancher

```shell
$ sudo docker run --privileged -d 
--restart=unless-stopped -p 80:80 -p 443:443 
-v <主机路径>:/var/lib/rancher/ rancher/rancher:stable
```



![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1gnc1axhh4tj319q15042v.jpg)

# 问题

## 1 安装cluster docker 联接不上

```shell
INFO[03:14:38 UTC] Generating etcd certs
INFO[03:14:41 UTC] Synchronizing etcd certs
ERRO[03:14:41 UTC] Failed to connect to 192.168.199.177: could not establish connection to 192.168.199.177:22: ssh: handshake failed: EOF  node=192.168.199.177
WARN[03:14:41 UTC] Task failed ...
WARN[03:14:41 UTC] error: interrupted by error
Error: Failed to sync etcd certs: interrupted by error
Usage:
  kk create cluster [flags]

Flags:
  -f, --filename string          Path to a configuration file
  -h, --help                     help for cluster
      --skip-pull-images         Skip pre pull images
      --with-kubernetes string   Specify a supported version of kubernetes
      --with-kubesphere          Deploy a specific version of kubesphere (default v3.0.0)
  -y, --yes                      Skip pre-check of the installation

Global Flags:
      --debug   Print detailed information (default true)
```

> 网络不通，重新运行一下

## 2 安装 kubesphere 报错默认存储未找到



```shell
fatal: [localhost]: FAILED! => {
    "assertion": "\"(default)\" in default_storage_class_check.stdout",
    "changed": false,
    "evaluated_to": false,
    "msg": "Default StorageClass was not found !"
}

```

> 在 cluster-configuration.yaml 文件添加 default StorageClass 类

## 3 重启 pod



```shell
$ kubectl rollout restart deploy -n kubesphere-system ks-installer
```

