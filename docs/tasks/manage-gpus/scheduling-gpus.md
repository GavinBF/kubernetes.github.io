---
assignees:
- vishh
title: ���� GPU
redirect_from:
- "/docs/user-guide/gpus/"
- "/docs/user-guide/gpus.html"
---
<!--
---
assignees:
- vishh
title: Schedule GPUs
redirect_from:
- "/docs/user-guide/gpus/"
- "/docs/user-guide/gpus.html"
---
-->

{% capture overview %}

<!--
Kubernetes includes **experimental** support for managing NVIDIA GPUs spread across nodes.
This page describes how users can consume GPUs and the current limitations.
-->
Kubernetes �ṩ�Էֲ��ڽڵ��ϵ� NVIDIA GPU ���й����**ʵ��**֧�֡���ҳ�����û����ʹ�� GPU �Լ���ǰʹ�õ�һЩ����

{% endcapture %}

{% capture prerequisites %}

<!--
1. Kubernetes nodes have to be pre-installed with Nvidia drivers. Kubelet will not detect Nvidia GPUs otherwise. Try to re-install nvidia drivers if kubelet fails to expose Nvidia GPUs as part of Node Capacity.
-->
1. Kubernetes �ڵ����Ԥ�Ȱ�װ�� NVIDIA ����������Kubelet ����ⲻ�����õ�GPU��Ϣ������ڵ�� Capacity ������û�г��� NIVIDA GPU ���������п���������û�а�װ���߰�װʧ�ܣ��볢�����°�װ
<!--
2. A special **alpha** feature gate `Accelerators` has to be set to true across the system: `--feature-gates="Accelerators=true"`.
-->
2. ������ Kubernetes ϵͳ�У�feature-gates �����ض��� **alpha** ���Բ��� `Accelerators` ��������Ϊ true��`--feature-gates="Accelerators=true"`
<!--
3. Nodes must be using `docker engine` as the container runtime.
-->
3. Kuberntes �ڵ����ʹ�� `docker` ������Ϊ��������������

<!--
The nodes will automatically discover and expose all Nvidia GPUs as a schedulable resource.
-->
����Ԥ��������ɺ󣬽ڵ���Զ������������ NVIDIA GPU����������Ϊ�ɵ�����Դ��¶

{% endcapture %}

{% capture steps %}

## API

<!--
Nvidia GPUs can be consumed via container level resource requirements using the resource name `alpha.kubernetes.io/nvidia-gpu`.
-->
��������ͨ������Ϊ `alpha.kubernetes.io/nvidia-gpu` �ı�ʶ��������Ҫʹ�õ� NVIDIA GPU ������

```yaml
apiVersion: v1
kind: pod
spec: 
  containers: 
    - 
      name: gpu-container-1
      resources: 
        limits: 
          alpha.kubernetes.io/nvidia-gpu: 2 # requesting 2 GPUs
    - 
      name: gpu-container-2
      resources: 
        limits: 
          alpha.kubernetes.io/nvidia-gpu: 3 # requesting 3 GPUs
```

<!--
- GPUs can be specified in the `limits` section only.
-->
- GPU ֻ����������Դ�� `limits` ������
<!--
- Containers (and pods) do not share GPUs.
-->
- ������ Pod ����֧�ֹ��� GPU
<!--
- Each container can request one or more GPUs.
-->
- ÿ��������������ʹ��һ�����߶�� GPU
<!--
- It is not possible to request a portion of a GPU.
-->
- GPU ����������Ϊ��λ������ʹ��
<!--
- Nodes are expected to be homogenous, i.e. run the same GPU hardware.
-->
- ���нڵ�� GPU Ӳ��Ҫ����ͬ

<!--
If your nodes are running different versions of GPUs, then use Node Labels and Node Selectors to schedule pods to appropriate GPUs.
Following is an illustration of this workflow:
-->
����ڲ�ͬ�Ľڵ����氲װ�˲�ͬ�汾�� GPU������ͨ�����ýڵ��ǩ�Լ�ʹ�ýڵ�ѡ�����ķ�ʽ�� pod ���ȵ��������еĽڵ��ϡ������������£�

<!--
As part of your Node bootstrapping, identify the GPU hardware type on your nodes and expose it as a node label.
-->
�ڽڵ��ϣ�ʶ��� GPU Ӳ�����ͣ�Ȼ������Ϊ�ڵ��ǩ���б�¶

```shell
NVIDIA_GPU_NAME=$(nvidia-smi --query-gpu=gpu_name --format=csv,noheader --id=0)
source /etc/default/kubelet
KUBELET_OPTS="$KUBELET_OPTS --node-labels='alpha.kubernetes.io/nvidia-gpu-name=$NVIDIA_GPU_NAME'"
echo "KUBELET_OPTS=$KUBELET_OPTS" > /etc/default/kubelet
```

<!--
Specify the GPU types a pod can use via [Node Affinity](/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity) rules.
-->
�� pod �ϣ�ͨ���ڵ�[�׺���](/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity)����Ϊ��ָ������ʹ�õ� GPU ����

```yaml
kind: pod
apiVersion: v1
metadata:
  annotations:
    scheduler.alpha.kubernetes.io/affinity: >
      {
        "nodeAffinity": {
          "requiredDuringSchedulingIgnoredDuringExecution": {
            "nodeSelectorTerms": [
              {
                "matchExpressions": [
                  {
                    "key": "alpha.kubernetes.io/nvidia-gpu-name",
                    "operator": "In",
                    "values": ["Tesla K80", "Tesla P100"]
                  }
                ]
              }
            ]
          }
        }
      }
spec: 
  containers: 
    - 
      name: gpu-container-1
      resources: 
        limits: 
          alpha.kubernetes.io/nvidia-gpu: 2
```

<!--
This will ensure that the pod will be scheduled to a node that has a `Tesla K80` or a `Tesla P100` Nvidia GPU.
-->
�����趨����ȷ�� pod �ᱻ���ȵ���������Ϊ `alpha.kubernetes.io/nvidia-gpu-name` �ı�ǩ���ұ�ǩ��ֵΪ `Tesla K80` ���� `Tesla P100` �Ľڵ���

<!--
### Warning
-->
### ����

<!--
The API presented here **will change** in an upcoming release to better support GPUs, and hardware accelerators in general, in Kubernetes.
-->
��δ���� Kubernetes �汾�ܹ����õ�֧��GPU�Լ�һ���Ӳ��������ʱ������� API ����**������֮�������**

<!--
## Access to CUDA libraries
-->
## ���� CUDA ��

<!--
As of now, CUDA libraries are expected to be pre-installed on the nodes.
-->
��ĿǰΪֹ������ҪԤ���ڽڵ��ϰ�װ CUDA ��

<!--
To mitigate this, you can copy the libraries to a more permissive folder in ``/var/lib/`` or change the permissions directly. (Future releases will automatically perform this operation)
-->
Ϊ�˱������ʹ�ÿ�������⣬���Խ���ŵ� ``/var/lib/`` �µ�ĳ���ļ����£�����ֱ�Ӹı��Ŀ¼��Ȩ��(�Ժ�İ汾���Զ������һ����)

<!--
Pods can access the libraries using `hostPath` volumes.
-->
Pods�ܹ�ͨ�� `hostPath` �������ʿ�

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: gpu-pod
spec:
  containers:
  - name: gpu-container-1
    securityContext:
      privileged: true
    resources:
      limits:
        alpha.kubernetes.io/nvidia-gpu: 1
    volumeMounts:
    - mountPath: /usr/local/nvidia/bin
      name: bin
    - mountPath: /usr/lib/nvidia
      name: lib
  volumes:
  - hostPath:
      path: /usr/lib/nvidia-375/bin
    name: bin
  - hostPath: 
      path: /usr/lib/nvidia-375
    name: lib
```

<!--
## Future
-->
## δ��

<!--
- Support for hardware accelerators is in it's early stages in Kubernetes.
-->
- Kubernetes ��Ӳ����������֧�ֻ��������ڽ׶�
<!--
- GPUs and other accelerators will soon be a native compute resource across the system.
-->
- GPU �������ļ������ܿ���Ϊϵͳ�ı��ؼ�����Դ
<!--
- Better APIs will be introduced to provision and consume accelerators in a scalable manner.
-->
- ��������õ� API �Կ���չ�ķ�ʽ�ṩ��ʹ�ü�����
<!--
- Kubernetes will automatically ensure that applications consuming GPUs gets the best possible performance.
-->
- Kubernets �����Զ�ȷ��Ӧ����ʹ�� GPU ʱ�õ��������
<!--
- Key usability problems like access to CUDA libraries will be addressed.
-->
- ���Ʒ��� CUDA �����ֹؼ��Ŀ��������⽫�õ����

{% endcapture %}

{% include templates/task.md %}
