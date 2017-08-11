---
assignees:
- caseydavenport
title: Ϊ�� NetworkPolicy ʹ�� Calico
redirect_from:
- "/docs/getting-started-guides/network-policy/calico/"
- "/docs/getting-started-guides/network-policy/calico.html"
- "/docs/tasks/configure-pod-container/calico-network-policy/"
- "/docs/tasks/configure-pod-container/calico-network-policy.html"
---
<!--
---
assignees:
- caseydavenport
title: Use Calico for NetworkPolicy
redirect_from:
- "/docs/getting-started-guides/network-policy/calico/"
- "/docs/getting-started-guides/network-policy/calico.html"
- "/docs/tasks/configure-pod-container/calico-network-policy/"
- "/docs/tasks/configure-pod-container/calico-network-policy.html"
---
-->

<!--
This page shows how to use Calico for NetworkPolicy.
-->
{% capture overview %}
��ҳչʾ��ô��Ϊ�� NetworkPolicy ʹ�� Calico
{% endcapture %}

<!--
* Install Calico for Kubernetes. 
-->
{% capture prerequisites %}
* Ϊ Kubernetes ��װ Calico
{% endcapture %}

{% capture steps %}
<!--
## Deploying a cluster using Calico
-->
## ʹ�� Calico ����һ����Ⱥ

<!--
You can deploy a cluster using Calico for network policy in the default [GCE deployment](/docs/getting-started-guides/gce) using the following set of commands:
-->
ʹ�����������������Ĭ�ϵ� [GCE](/docs/getting-started-guides/gce) �ϲ����һ��Ϊ�������ʹ�� Calico �ļ�Ⱥ������

```shell
export NETWORK_POLICY_PROVIDER=calico
export KUBE_NODE_OS_DISTRIBUTION=debian
curl -sS https://get.k8s.io | bash
```

<!--
See the [Calico documentation](http://docs.projectcalico.org/) for more options to deploy Calico with Kubernetes.
-->
����Ĳ���ѡ����ο� [Calico ��Ŀ�ĵ�](http://docs.projectcalico.org/)
{% endcapture %}

{% capture discussion %}
<!--
##  Understanding Calico components
-->
##  ��� Calico ���

<!--
Deploying a cluster with Calico adds Pods that support Kubernetes NetworkPolicy.  These Pods run in the `kube-system` Namespace. 
-->
����ʹ�� Calico �ļ�Ⱥ��ʵ��������֧�� Kubernetes NetworkPolicy �� Pods�� ��Щ Pods ������ `kube-system` �����ռ��¡�

<!--
To see this list of Pods run:
-->
ʹ�����·�ʽȥ�鿴��Щ���е� Pods��

```shell
kubectl get pods --namespace=kube-system
```

<!--
You'll see a list of Pods similar to this:
-->
�����Կ�����������������һ�� Pods �б�

```console
NAME                                                 READY     STATUS    RESTARTS   AGE
calico-node-kubernetes-minion-group-jck6             1/1       Running   0          46m
calico-node-kubernetes-minion-group-k9jy             1/1       Running   0          46m
calico-node-kubernetes-minion-group-szgr             1/1       Running   0          46m
calico-policy-controller-65rw1                       1/1       Running   0          46m
...
```

<!--
There are two main components to be aware of:
-->
��Ҫ���������

<!--
- One `calico-node` Pod runs on each node in your cluster and enforces network policy on the traffic to/from Pods on that machine by configuring iptables.
-->
- �ڼ�Ⱥ��ÿ���ڵ��϶�������һ���� `calico-node` ��ͷ������ Pod���������� iptables ȥʵ����Щ������ Pods �ĳ�/���������
<!--
- The `calico-policy-controller` Pod reads the policy and label information from the Kubernetes API and configures Calico appropriately.
-->
- ������Ⱥ����ֻ��һ���� `calico-policy-controller` ��ͷ������ Pod�����ڴ� Kubernetes API �ж�ȡ���Ժͱ�ǩ��Ϣ���ʵ��Ķ� Calico ��������
{% endcapture %}

<!--
Once your cluster is running, you can follow the [NetworkPolicy getting started guide](/docs/getting-started-guides/network-policy/walkthrough) to try out Kubernetes NetworkPolicy.
-->
{% capture whatsnext %}
��Ⱥ�������֮��������ͨ�� [NetworkPolicy ����ָ��](/docs/getting-started-guides/network-policy/walkthrough)ȥ����ʹ�� Kubernetes NetworkPolicy
{% endcapture %}

{% include templates/task.md %}
