---
assignees:
- jbeda
<!-- title: Authenticating with Bootstrap Tokens -->
title: Webhook 模式
---

* TOC
{:toc}

<!-- ## Overview -->
## 概述

<!-- Bootstrap tokens are a simple bearer token that is meant to be used when -->
<!-- creating new clusters or joining new nodes to an existing cluster.  It was built -->
<!-- to support [`kubeadm`](/docs/admin/kubeadm/), but can be used in other contexts -->
<!-- for users that wish to start clusters without `kubeadm`. It is also built to -->
<!-- work, via RBAC policy, with the [Kubelet TLS -->
<!-- Bootstrapping](/docs/admin/kubelet-tls-bootstrapping/) system. -->
启动引导令牌是一种简单的 bearer token ，这种令牌是在新建集群或者在现有集群中添加新加新节点时使用的。
它被设计成能支持 [`kubeadm`](/docs/admin/kubeadm/)，但是也可以被用在其他上下文中以便用户在
不使用 `kubeadm` 的情况下启动cluster。它也被设计成可以通过 RBAC 策略，结合[Kubelet TLS
Bootstrapping](/docs/admin/kubelet-tls-bootstrapping/) 系统进行工作。

<!-- Bootstrap Tokens are defined with a specific type -->
<!-- (`bootstrap.kubernetes.io/token`) of secrets that lives in the `kube-system` -->
<!-- namespace.  These Secrets are then read by the Bootstrap Authenticator in the -->
<!-- API Server.  Expired tokens are removed with the TokenCleaner controller in the -->
<!-- Controller Manager.  The tokens are also used to create a signature for a -->
<!-- specific ConfigMap used in a "discovery" process through a BootstrapSigner -->
<!-- controller. -->
启动引导令牌被定义成一个特定类型的secrets (`bootstrap.kubernetes.io/token`)，并存在于
`kube-system` 命名空间中。然后这些 secrets 会被 API 服务器上的启动引导的认证器读取。
过期的令牌与令牌清除控制器会被控制管理器一起清除。令牌也会被用于创建签名，签名用于
启动引导签名控制器在 "discovery" 进程中特定的 configmap 。


<!-- Currently, Bootstrap Tokens are **alpha** but there are no large breaking -->
<!-- changes expected. -->
目前，启动引导令牌处于 **alpha** 阶段，但是预期也不会有大的突破性的变化。

<!-- ## Token Format -->
## 令牌格式

<!-- Bootstrap Tokens take the form of `abcdef.0123456789abcdef`.  More formally, -->
<!-- they must match the regular expression `[a-z0-9]{6}\.[a-z0-9]{16}`. -->
启动引导令牌使用 `abcdef.0123456789abcdef` 的形式。
更加规范地说，它们必须符合正则表达式 `[a-z0-9]{6}\.[a-z0-9]{16}`。

<!-- The first part of the token is the "Token ID" and is considered public -->
<!-- information.  It is used when referring to a token without leaking the secret -->
<!-- part used for authentication. The second part is the "Token Secret" and should -->
<!-- only be shared with trusted parties. -->
令牌的第一部分是 "Token ID" ，它是公共信息。它被用于引用一个 token 用于认证而不会泄漏保密部分。
第二部分是 "Token Secret"，它应该只能被信任方共享。

<!-- ## Enabling Bootstrap Tokens -->
## 启用启动引导令牌

<!-- All features for Bootstrap Tokens are disabled by default in Kubernetes v1.6. -->
所有启动引导令牌的特新在 Kubernetes v1.6 版本中都是默认禁用的。

<!-- You can enable the Bootstrap Token authenticator with the -->
<!-- `--experimental-bootstrap-token-auth` flag on the API server.  You can enable -->
<!-- the Bootstrap controllers by specifying them withthe `--controllers` flag on the -->
<!-- controller manager with something like -->
<!-- `--controllers=*,tokencleaner,bootstrapsigner`.  This is done automatically when -->
<!-- using `kubeadm`. -->
你可以在 API 服务器上通过 `--experimental-bootstrap-token-auth` 参数启用启动引导令牌。
你可以在控制管理器上通过 `--controllers` 参数，比如 `--controllers=*,tokencleaner,bootstrapsigner` 来启用启动引导令牌。
在使用 `kubeadm` 时，这个是自动完成的。

<!-- Tokens are used in an HTTPS call as follows: -->
HTTPS 调用中的令牌是这样使用的：

```http
Authorization: Bearer 07401b.f395accd246ae52d
```

<!-- ## Bootstrap Token Secret Format -->
## 启动引导令牌的密文格式

<!-- Each valid token is backed by a secret in the `kube-system` namespace.  You can -->
<!-- find the full design doc -->
<!-- [here](https://git.k8s.io/community/contributors/design-proposals/bootstrap-discovery.md). -->
每个合法的令牌是通过一个 `kube-system` 命名空间中的 secret 隐藏的。
你可以从[这里](https://git.k8s.io/community/contributors/design-proposals/bootstrap-discovery.md)找到完整设计文档。

<!-- Here is what the secret looks like.  Note that `base64(string)` indicates the -->
<!-- value should be base64 encoded.  The undecoded version is provided here for -->
<!-- readability. -->
这是 secret 看起来的样子。注意，`base64(string)` 表示值应该是通过 base64 编码的。
这里使用的是解码版本以便于阅读。

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: bootstrap-token-07401b
  namespace: kube-system
type: bootstrap.kubernetes.io/token
data:
  description: base64(The default bootstrap token generated by 'kubeadm init'.)
  token-id: base64(07401b)
  token-secret: base64(f395accd246ae52d)
  expiration: base64(2017-03-10T03:22:11Z)
  usage-bootstrap-authentication: base64(true)
  usage-bootstrap-signing: base64(true)
```

<!-- The type of the secret must be `bootstrap.kubernetes.io/token` and the name must -->
<!-- be `bootstrap-token-<token id>`.  It must also exist in the `kube-system` -->
<!-- namespace.  `description` is a human readable discription that should not be -->
<!-- used for machine readable information.  The Token ID and Secret are included in -->
<!-- the data dictionary. -->
secret的类型必须是 `bootstrap.kubernetes.io/token` ，而名字必须是 `bootstrap-token-<token id>`。
`description` 是人类可读的描述，而不应该是机器可读的信息。令牌 ID 和 Secret 是包含在数据字典中的。

The `usage-bootstrap-*` members indicate what this secret is intended to be used
for.  A value must be set to `true` to be enabled.
 `usage-bootstrap-*` 成员代表了这个 secret 被用于什么。启用时，值必须设置为 `true`。

<!-- `usage-bootstrap-authentication` indicates that the token can be used to -->
<!-- authenticate to the API server.  The authenticator authenticates as -->
<!-- `system:bootstrap:<Token ID>`.  It is included in the `system:bootstrappers` -->
<!-- group.  The naming and groups are intentionally limited to discourage users from -->
<!-- using these tokens past bootstrapping. -->
`usage-bootstrap-authentication` 表示令牌可以用于 API 服务器的认证。认证器会以
`system:bootstrap:<Token ID>` 认证。它被包含在 `system:bootstrappers` 组中。
命名和组是故意受限制的，以阻碍用户在启动引导后再使用这些令牌。

<!-- `usage-bootstrap-signing` indicates that the token should be used to sign the -->
<!-- `cluster-info` ConfigMap as described below. -->
`usage-bootstrap-signing` 表示令牌应该被用于 `cluster-info` ConfigMap 的签名，就像下面描述的那样。

<!-- The `expiration` data member lists a time after which the token is no longer -->
<!-- valid.  This is encoded as an absolute UTC time using RFC3339.  The TokenCleaner -->
<!-- controller will delete expired tokens. -->
`expiration` 数据成员列举了令牌在失效后的时间。这是通过 RFC3339 进行编码的 UTC 时间。
令牌清理控制器会删除过期的令牌。

<!-- ## Token Management with `kubeadm` -->
## 使用 `kubeadm` 管理令牌

<!-- You can use the `kubeadm` tool to manage tokens on a running cluster.  It will -->
<!-- automatically grab the default admin credentials on a master from a `kubeadm` -->
<!-- created cluster (`/etc/kubernetes/admin.conf`).  You can specify an alternate -->
<!-- kubeconfig file for credentials with the `--kubeconfig` to the following -->
<!-- commands. -->
你可以是用 `kubeadm` 工具管理正在运行集群的令牌。它会从 `kubeadm` 创建的集群(`/etc/kubernetes/admin.conf`)
自动抓取默认管理员密码。你可以对下面命令指定一个另外的 kubeconfig 文件抓取密码，参数使用 `--kubeconfig`。

<!-- * `kubeadm token list` Lists the tokens along with when they expire and what the
  approved usages are.
* `kubeadm token create` Creates a new token.
    * `--description` Set the description on the new token.
    * `--ttl duration` Set expiration time of the token as a delta from "now".
      Default is 0 for no expiration.
    * `--usages` Set the ways that the token can be used.  The default is
      `signing,authentication`.  These are the usages as described above.
* `kubeadm token delete <token id>|<token id>.<token secret>` Delete a token.
  The token can either be identified with just an ID or with the entire token
  value.  Only the ID is used; the token is still deleted if the secret does not
  match. -->
* `kubeadm token list` 列举了令牌，同时显示了它们的过期时间和用途。
* `kubeadm token create` 创建一个新令牌。
    * `--description` 设置新令牌的描述。
    * `--ttl duration` 设置令牌从 "现在" 算起到过期的时间增量。
      默认是 0 ，也就是不过期。
    * `--usages` 设置令牌被使用的方式。默认是 `signing,authentication`。用途在上面已经描述。
* `kubeadm token delete <token id>|<token id>.<token secret>` 删除令牌。
  令牌可以只用 ID 来确认，或者用整个令牌的值。如果只用 ID，密文不符合的令牌也会被删除。

<!-- ## ConfigMap Signing -->
### ConfigMap签名

<!-- In addition to authentication, the tokens can be used to sign a ConfigMap.  This -->
<!-- is used early in a cluster bootstrap process before the client trusts the API -->
<!-- server.  The signed ConfigMap can be authenicated by the shared token. -->
除了认证之外，令牌可以用于签名 ConfigMap。这在集群启动流程的早期，在客户端信任 API服务器之前被使用。
签名过的 ConfigMap 可以通过共享令牌被认证。

<!-- The ConfigMap that is signed is `cluster-info` in the `kube-public` namespace. -->
<!-- The typical flow is that a client reads this ConfigMap while unauthenticated and -->
<!-- ignoring TLS errors.  It then validates the payload of the ConfigMap by looking -->
<!-- at a signature embedded in the ConfigMap. -->
签名过的 ConfigMap 是 `kube-public` 命名空间中的 `cluster-info`。
典型的工作流中，客户端读取这个 ConfigMap 而不管认证和 TLS 报错。
它会通过 ConfigMap 中嵌入的签名校验 ConfigMap 的载荷。

<!-- The ConfigMap may look like this: -->
ConfigMap 会是这个样子的：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-info
  namespace: kube-public
data:
  jws-kubeconfig-07401b: eyJhbGciOiJIUzI1NiIsImtpZCI6IjA3NDAxYiJ9..tYEfbo6zDNo40MQE07aZcQX2m3EB2rO3NuXtxVMYm9U
  kubeconfig: |
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: <really long certificate data>
        server: https://10.138.0.2:6443
      name: ""
    contexts: []
    current-context: ""
    kind: Config
    preferences: {}
    users: []
```

<!-- The `kubeconfig` member of the ConfigMap is a config file with just the cluster -->
<!-- information filled out.  The key thing being communicated here is the -->
<!-- `certificate-authority-data`.  This may be expanded in the future. -->
ConfigMap 的 `kubeconfig` 成员是一个填好了集群信息的配置文件。
这里主要交换的信息是 `certificate-authority-data`。在将来可能会被扩展。

<!-- The signature is a JWS signature using the "detached" mode.  To validate the -->
<!-- signature, the user should encode the `kubeconfig` payload according to JWS -->
<!-- rules (base64 encoded while discarding any trailing `=`).  That encoded payload -->
<!-- is then used to form a whole JWS by inserting it between the 2 dots.  You can -->
<!-- verify the JWS using the `HS256` scheme (HMAC-SHA256) with the full token (e.g. -->
<!-- `07401b.f395accd246ae52d`) as the shared secret.  Users _must_ verify that HS256 -->
<!-- is used. -->
签名是一个 JWS 签名，使用了 "detached" 模式。为了检验签名，用户应该按照 JWS 规则
(base64 编码而忽略结尾的 `=`)对 `kubeconfig` 载荷进行编码。那样编码过载荷会被通过插入 JWS 并存在于两个点的中间
，用于形成一个完整的 JWS。你可以使用令牌的完整信息(比如 `07401b.f395accd246ae52d`)作为共享密钥，
通过 `HS256` 方式 (HMAC-SHA256) 对 JWS 进行校验。 用户 _必须_ 确保使用了 HS256。