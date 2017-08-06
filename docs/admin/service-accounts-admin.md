---
assignees:
- bprashanth
- davidopp
- lavalamp
- liggitt
title: Managing Service Accounts
---
<!--
*This is a Cluster Administrator guide to service accounts.  It assumes knowledge of
the [User Guide to Service Accounts](/docs/user-guide/service-accounts).*

*Support for authorization and user accounts is planned but incomplete.  Sometimes
incomplete features are referred to in order to better describe service accounts.*
-->

*本文是 service accounts 管理员手册。需要[Service Accounts 用户手册](/docs/user-guide/service-accounts)的知识背景。*

*对于 authorization 和 user accounts 支持已经在计划中，但是还未完成。为了更好地解释 service
accounts，有时会提到一些未完成的功能特性。*

## User accounts vs service accounts
<!--
Kubernetes distinguished between the concept of a user account and a service accounts
for a number of reasons:

  - User accounts are for humans.  Service accounts are for processes, which
    run in pods.
  - User accounts are intended to be global. Names must be unique across all
    namespaces of a cluster, future user resource will not be namespaced.
    Service accounts are namespaced.
  - Typically, a cluster's User accounts might be synced from a corporate
    database, where new user account creation requires special privileges and
    is tied to complex business  processes.  Service account creation is intended
    to be more lightweight, allowing cluster users to create service accounts for
    specific tasks (i.e. principle of least privilege).
  - Auditing considerations for humans and service accounts may differ.
  - A config bundle for a complex system may include definition of various service
    accounts for components of that system.  Because service accounts can be created
    ad-hoc and have namespaced names, such config is portable.
-->

Kubernetes 将 user account 和 service account 的概念区分开，主要基于以下几个原因：

## Service account automation
<!--
Three separate components cooperate to implement the automation around service accounts:

  - A Service account admission controller
  - A Token controller
  - A Service account controller
-->

service accounts 的自动化由三个独立的组建共同配合实现：

  - A Service account admission controller (service account 准入控制器)
  - A Token controller (令牌控制器)
  - A Service account controller (service account 控制器)

### Service Account Admission Controller
<!--
The modification of pods is implemented via a plugin
called an [Admission Controller](/docs/admin/admission-controllers). It is part of the apiserver.
It acts synchronously to modify pods as they are created or updated. When this plugin is active
(and it is by default on most distributions), then it does the following when a pod is created or modified:

  1. If the pod does not have a `ServiceAccount` set, it sets the `ServiceAccount` to `default`.
  2. It ensures that the `ServiceAccount` referenced by the pod exists, and otherwise rejects it.
  4. If the pod does not contain any `ImagePullSecrets`, then `ImagePullSecrets` of the
`ServiceAccount` are added to the pod.
  5. It adds a `volume` to the pod which contains a token for API access.
  6. It adds a `volumeSource` to each container of the pod mounted at `/var/run/secrets/kubernetes.io/serviceaccount`.
-->

对于 pods 的操作是通过一个叫做 [准入控制器](/docs/admin/admission-controllers) 的插件实现的。它是 apiserver 的一部分。
当准入控制器被创建或更新时，他会对 pod 同步进行操作。当这个插件时活动状态时（大部分版本默认是活动状态），并在 pod 被创建或者更改时，
它会做如下操作：

 1. 如果 pod 没有配置 `ServiceAccount`，它会将 `ServiceAccount` 设置为  `default`。
 2. 它会确保被 pod 关联的 `ServiceAccount` 是存在的，否则就拒绝请求。
 <!-- 这里为什么没有3，直接跳到4了 -->
 4. 如果 pod 没有包含任何的 `ImagePullSecrets`，那么 `ServiceAccount` 的 `ImagePullSecrets` 就会被添加到 pod。
 5. 它会把一个 `volume` 添加给 pod， 该 pod 包含有一个用于 API 访问的 token。
 6. 它会把一个 `volumeSource` 添加到 pod 的每个 container，并挂载到 `/var/run/secrets/kubernetes.io/serviceaccount`。

### Token Controller
<!--
TokenController runs as part of controller-manager. It acts asynchronously. It:

- observes serviceAccount creation and creates a corresponding Secret to allow API access.
- observes serviceAccount deletion and deletes all corresponding ServiceAccountToken Secrets
- observes secret addition, and ensures the referenced ServiceAccount exists, and adds a token to the secret if needed
- observes secret deletion and removes a reference from the corresponding ServiceAccount if needed

You must pass a service account private key file to the token controller in the controller-manager by using
the `--service-account-private-key-file` option. The private key will be used to sign generated service account tokens.
Similarly, you must pass the corresponding public key to the kube-apiserver using the `--service-account-key-file`
option. The public key will be used to verify the tokens during authentication.
-->

令牌控制器（TokenController）作为 controller-manager 的一部分运行。它异步运行。它会：

- 监听对于 serviceAccount 的创建动作，并创建对应的 Secret 以允许 API 访问。
- 监听对于 serviceAccount 的删除动作，并删除所有对应的 ServiceAccountToken Secrets。
- 监听对于 secret 的添加动作，确保相关联的 ServiceAccount 是存在的，并在根据需要为 secret 添加一个 token。
- 监听对于 secret 的删除动作，并根据需要删除对应 ServiceAccount 的关联。

你必须给令牌控制器（token controller）传递一个 service account 的私钥（private key），可以通过 `--service-account-private-key-file` 参数完成。
传递的私钥将被用来对 service account tokens 进行签名。类似的，你必须给 kube-apiserver 传递一个公钥（public key），通过 `--service-account-key-file`
参数完成。传递的公钥会被用来验证认证过程中的令牌（token）。

#### To create additional API tokens
<!--
A controller loop ensures a secret with an API token exists for each service
account. To create additional API tokens for a service account, create a secret
of type `ServiceAccountToken` with an annotation referencing the service
account, and the controller will update it with a generated token:
-->

控制器的循环运行会确保对于每个 service account 都存在一个带有 API token（API 令牌）的secret。
如需要为一个 service account 创建一个额外的 API 令牌（API token），可以创建一个 `ServiceAccountToken`
类型的 secret，并添加与 service account 对应的 annotation 属性，控制器会为它更新 token：

secret.json:

```json
{
    "kind": "Secret",
    "apiVersion": "v1",
    "metadata": {
        "name": "mysecretname",
        "annotations": {
            "kubernetes.io/service-account.name": "myserviceaccount"
        }
    },
    "type": "kubernetes.io/service-account-token"
}
```

```shell
kubectl create -f ./secret.json
kubectl describe secret mysecretname
```

#### To delete/invalidate a service account token

```shell
kubectl delete secret mysecretname
```

### Service Account Controller
<!--
Service Account Controller manages ServiceAccount inside namespaces, and ensures
a ServiceAccount named "default" exists in every active namespace.
-->
Service Account Controller 在 namespaces 内管理 ServiceAccount，需要保证名为 "default" 的
ServiceAccount在每个命名空间中存在。