---
id: tsg_install
title: Troubleshooting OpenEBS Installation
sidebar_label: Installation
---

------

## On Azure, stateful pods are not running when OpenEBS volumes are provisioned

On AKS, after provisioning the OpenEBS volume through the OpenEBS dynamic provisioner, the application pod is not coming up. It stays in `ContainerCreating` state

```
kubectl get pods
NAME                                                            READY     STATUS              RESTARTS   AGE
maya-apiserver-7b8f548dd8-67s6x                                 1/1       Running             0          36m
openebs-provisioner-7958c6d44f-g9qvr                            1/1       Running             0          36m
pgset-0                                                         0/1       ContainerCreating   0          32m
pvc-febcc15e-25d7-11e8-92c2-0a58ac1f1190-ctrl-7d7c98745-49qcm   2/2       Running             0          32m
pvc-febcc15e-25d7-11e8-92c2-0a58ac1f1190-rep-578b5bcc6b-5758m   1/1       Running             0          32m
pvc-febcc15e-25d7-11e8-92c2-0a58ac1f1190-rep-578b5bcc6b-zkhn8   1/1       Running             0          32m
```

The AKS cluster runs ubuntu 16.04 LTS with the kubelet running in a container (debian-jessie 8). The kubelet logs show the absence of the iSCSI initiator. Hence, the volume is not attached to the node. Configuring kubelet to run with iSCSI utils should fix this issue. 

This steps are provided in pre-requisite section, see [here](/docs/next/prerequisites.html#azure-cloud) 

For more information, see  https://github.com/openebs/openebs/issues/1335.

## On Azure, OpenEBS installation fails

On AKS, while installing OpenEBS using Helm, if you get the following error, you must enable RBAC on Azure. For more details, see [Prerequisites](/docs/next/prerequisites.html).

```
$ helm installstable/openebs --name openebs --namespace openebs
Error: release openebsfailed: clusterroles.rbac.authorization.k8s.io "openebs" isforbidden: attempt to grant extra privileges:[PolicyRule{Resources:["nodes"], APIGroups:["*"],Verbs:["get"]} PolicyRule{Resources:["nodes"],APIGroups:["*"], Verbs:["list"]}PolicyRule{Resources:["nodes"], APIGroups:["*"],Verbs:["watch"]} PolicyRule{Resources:["nodes/proxy"],APIGroups:["*"], Verbs:["get"]}PolicyRule{Resources:["nodes/proxy"], APIGroups:["*"],Verbs:["list"]} PolicyRule{Resources:["nodes/proxy"],APIGroups:["*"], Verbs:["watch"]}PolicyRule{Resources:["namespaces"], APIGroups:["*"],Verbs:["*"]} PolicyRule{Resources:["services"],APIGroups:["*"], Verbs:["*"]} PolicyRule{Resources:["pods"],APIGroups:["*"], Verbs:["*"]}PolicyRule{Resources:["deployments"], APIGroups:["*"],Verbs:["*"]} PolicyRule{Resources:["events"],APIGroups:["*"], Verbs:["*"]}PolicyRule{Resources:["endpoints"], APIGroups:["*"],Verbs:["*"]} PolicyRule{Resources:["persistentvolumes"],APIGroups:["*"], Verbs:["*"]} PolicyRule{Resources:["persistentvolumeclaims"],APIGroups:["*"], Verbs:["*"]}PolicyRule{Resources:["storageclasses"],APIGroups:["storage.k8s.io"], Verbs:["*"]}PolicyRule{Resources:["storagepools"], APIGroups:["*"],Verbs:["get"]} PolicyRule{Resources:["storagepools"], APIGroups:["*"],Verbs:["list"]} PolicyRule{NonResourceURLs:["/metrics"],Verbs:["get"]}] user=&{system:serviceaccount:kube-system:tiller6f3172cc-4a08-11e8-9af5-0a58ac1f1729 [system:serviceaccounts system:serviceaccounts:kube-systemsystem:authenticated] map[]} ownerrules=[]ruleResolutionErrors=[clusterroles.rbac.authorization.k8s.io"cluster-admin" not found]
```

## On Rancher, application pods are not running when OpenEBS volumes are provisioned

The setup environment where the issue occurs is rancher/rke with bare metal hosts running CentOS. After installing OpenEBS, OpenEBS pods are running, but application pod is in *ContainerCreating* state. The output of `kubectl get pods` is displayed as follows.

```
NAME                                                             READY     STATUS              RESTARTS   AGE
nginx-deployment-57849d9f57-gvzkh                                0/1       ContainerCreating   0          2m
pvc-adb79406-8e3e-11e8-a06a-001c42c2325f-ctrl-58dcdf997f-n4kd9   2/2       Running             0          8m
pvc-adb79406-8e3e-11e8-a06a-001c42c2325f-rep-696b599894-gq4z6    1/1       Running             0          8m
pvc-adb79406-8e3e-11e8-a06a-001c42c2325f-rep-696b599894-hwx52    1/1       Running             0          8m
pvc-adb79406-8e3e-11e8-a06a-001c42c2325f-rep-696b599894-vs97n    1/1       Running             0          8m
```

iSCSI package is installed on both Host and rke kubelet.

```
[root@node-34622 ~]# iscsiadm -V
iscsiadm version 6.2.0.874-7
[root@node-34622 ~]# docker exec kubelet iscsiadm -V
iscsiadm version 2.0-874
```

To resolve this issue, do not install `open-iscsi / iscsi-initiator-utils` on the host nodes when using the Rancher Container Engine (RKE).

<!-- Hotjar Tracking Code for https://docs.openebs.io -->
<script>


```
   (function(h,o,t,j,a,r){
   		h.hj=h.hj||function(){(h.hj.q=h.hj.q||[]).push(arguments)};
   		h._hjSettings={hjid:785693,hjsv:6};
   		a=o.getElementsByTagName('head')[0];
   		r=o.createElement('script');r.async=1;
   		r.src=t+h._hjSettings.hjid+j+h._hjSettings.hjsv;
   		a.appendChild(r);
   })(window,document,'https://static.hotjar.com/c/hotjar-','.js?sv=');
```


</script>
