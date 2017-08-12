---
assignees:
- chrismarino
title: Ϊ�� NetworkPolicy ʹ�� Romana
redirect_from:
- "/docs/getting-started-guides/network-policy/romana/"
- "/docs/getting-started-guides/network-policy/romana.html"
- "/docs/tasks/configure-pod-container/romana-network-policy/"
- "/docs/tasks/configure-pod-container/romana-network-policy.html"
---
<!--
---
assignees:
- chrismarino
title: Romana for NetworkPolicy
redirect_from:
- "/docs/getting-started-guides/network-policy/romana/"
- "/docs/getting-started-guides/network-policy/romana.html"
- "/docs/tasks/configure-pod-container/romana-network-policy/"
- "/docs/tasks/configure-pod-container/romana-network-policy.html"
---
-->

{% capture overview %}

<!--
This page shows how to use Romana for NetworkPolicy.
-->
��ҳչʾ��ô��Ϊ�� NetworkPolicy ʹ�� Romana

{% endcapture %}

{% capture prerequisites %}

<!--
Complete steps 1, 2, and 3 of  the [kubeadm getting started guide](/docs/getting-started-guides/kubeadm/). 
-->
��� [kubeadm ����ָ��](/docs/getting-started-guides/kubeadm/)�еĲ���1��2��3

{% endcapture %}

{% capture steps %}

<!--
## Installing Romana with kubeadm
-->
## ʹ�� kubeadm ��װ Romana

<!--
Follow the [containerized installation guide](https://github.com/romana/romana/tree/master/containerize) for kubeadmin. 
-->
����[��������װָ��](https://github.com/romana/romana/tree/master/containerize)��ʹ�� kubeadm �ķ�ʽ��װ

<!--
## Applying network policies
-->
## Ӧ���������

<!--
To apply network policies use one of the following:
-->
ҪӦ��������ԣ���ʹ�����·�ʽ֮һ��

<!--
* [Romana network policies](https://github.com/romana/romana/wiki/Romana-policies). 
    * [Example of Romana network policy](https://github.com/romana/core/tree/master/policy).
-->
* [Romana �������](https://github.com/romana/romana/wiki/Romana-policies)
    * [Romana �������ʾ��](https://github.com/romana/core/tree/master/policy)
<!--
* The NetworkPolicy API.
-->
* NetworkPolicy API
 
{% endcapture %}

{% capture whatsnext %}

<!--
Once your have installed Romana, you can follow the [NetworkPolicy getting started guide](/docs/getting-started-guides/network-policy/walkthrough) to try out Kubernetes NetworkPolicy.
-->
Romana ��װ���֮��������ͨ�� [NetworkPolicy ����ָ��](/docs/getting-started-guides/network-policy/walkthrough)ȥ����ʹ�� Kubernetes NetworkPolicy

{% endcapture %}

{% include templates/task.md %}




