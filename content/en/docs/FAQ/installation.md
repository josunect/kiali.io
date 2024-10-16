---
title: "Installation"
description: "Questions about Kiali installation options or issues."
---

### Operator fails due to `cannot list resource "clusterroles"` error

When the Kiali Operator installs a Kiali Server, the Operator will assign the Kiali Server the proper roles/rolebindings so the Kiali Server can access the appropriate namespaces.

The Kiali Operator will check to see if the Kiali CR setting `deployment.cluster_wide_access` is set to `true` (which is the default value if it is unset). If it is, this means the Kiali Server is to be given access to all namespaces in the cluster, including namespaces that will be created in the future. In this case, the Kiali Operator will create and assign ClusterRole/ClusterRoleBinding resources to the Kiali Server. But in order to be able to do this, the Kiali Operator must itself be given permission to create those ClusterRole and ClusterRoleBinding resources. When you install the Kiali Operator via OLM, these permissions are automatically granted. However, if you installed the Kiali Operator with the [Operator Helm Chart](https://kiali.org/helm-charts/index.yaml), and if you did so with the value [`clusterRoleCreator`](https://github.com/kiali/helm-charts/blob/v1.88.0/kiali-operator/values.yaml#L41-L45)
set to `false` then the Kiali Operator will not be given permission to create cluster roles. In this case, you will be unable to install a Kiali Server if your Kiali CR does not have `deployment.cluster_wide_access` set to `true` (and, again, this is the default if unspecified). You will get an error similar to this:

```
Failed to list rbac.authorization.k8s.io/v1, Kind=ClusterRole:
clusterroles.rbac.authorization.k8s.io is forbidden:
User "system:serviceaccount:kiali-operator:kiali-operator"
cannot list resource "clusterroles" in API group
"rbac.authorization.k8s.io" at the cluster scope
```

Thus, if you do not give the Kiali Operator the permission to create cluster roles, you must tell the Operator which specific namespaces the Kiali Server can access. When specific namespaces are specified in `deployment.discovery_selectors.default`, the Kiali Operator will create Role and RoleBindings (not the "Cluster" kinds) and assign them to the Kiali Server.


### What values can be set in the Kiali CR?

A Kiali CR is used to tell the Kiali Operator how and where to install a Kiali Server in your cluster. You can install one or more Kiali Servers by creating
one Kiali CR for each Kiali Server you want the Operator to install and manage. Deleting a Kiali CR will uninstall its associted Kiali Server.

Most options are described in the pages of the [Installation]({{< relref "../Installation" >}}) and [Configuration]({{< relref "../Configuration" >}}) sections of the documentation.

If you cannot find some configuration, check the [Kiali CR Reference](/docs/configuration/kialis.kiali.io), which briefly describes all available options along with an example CR and all default values.
If you are using a specific version of the Operator prior to 1.46, the Kiali CR that is valid for that version can be found in the version tag within the github repository (e.g. Operator v1.25.0 supported [these Kiali CR settings](https://github.com/kiali/kiali-operator/blob/v1.25.0/deploy/kiali/kiali_cr.yaml)).

### How to configure some operator features at runtime {#operator-configuration}

{{% alert color="danger" %}}
First, read 
[Managing configuration of Helm installations in the Installation guide]({{< ref "/docs/installation/installation-guide/install-with-helm#managing-installation-config" >}}) to
check if that method works for your case.
{{% /alert %}}

Once the Kiali Operator is installed, you can change some of its configuration at runtime in order to utilize certain features that the Kiali Operator provides.
These features are configured via environment variables defined in the operator's deployment.

{{% alert color="warning" %}}
Only a user with admin permissions can configure these environment variables. You must make sure you know what you are doing before attempting to modify these environment variables.
Doing things incorrectly may break the Kiali Operator.
{{% /alert %}}

Perform the following steps to configure these features in the Kiali Operator:

1. Determine the namespace where your operator is located and store that namespace name in `$OPERATOR_NAMESPACE`. If you installed the operator via helm,
it may be `kiali-operator`. If you installed the operator via OLM, it may be `openshift-operators`. If you are not sure, you can perform a query to find it:

```
OPERATOR_NAMESPACE="$(kubectl get deployments --all-namespaces  | grep kiali-operator | cut -d ' ' -f 1)"
```

2. Determine the name of the environment variable you need to change in order to configure the feature you are interested in. Here is a list of currently supported environment variables you can set:

- `ALLOW_AD_HOC_KIALI_NAMESPACE`: must be `true` or `false`. If `true`, the operator will be allowed to install the Kiali Server in any namespace, regardless of which namespace the Kiali CR is created. If `false`, the operator will only install the Kiali Server in the same namespace where the Kiali CR is created - any attempt to do otherwise will cause the operator to abort the Kiali Server installation.
- `ALLOW_AD_HOC_KIALI_IMAGE`: must be `true` or `false`. If `true`, the operator will be allowed to install the Kiali Server with a custom container image as defined in the Kiali CR's `spec.deployment.image_name` and/or `spec.deployment.image_version`. If `false`, the operator will only install the Kiali Server with the default image. If a Kiali CR is created with `spec.deployment.image_name` or `spec.deployment.image_version` defined, the operator will abort the Kiali Server installation.
- `ALLOW_SECURITY_CONTEXT_OVERRIDE`: must be `true` or `false`. If `true`, the operator will be allowed to install the Kiali Server container with a fully customizable securityContext as defined by the user in the Kial CR. If `false`, the operator will only allow the user to add settings to the securityContext; any attempt to override the default settings in the securityContext will be ignored.
- `ALLOW_ALL_ACCESSIBLE_NAMESPACES`: must be `true` or `false`. If `true`, the operator will allow the user to configure Kiali to access all namespaces in the cluster (i.e. will allow a Kiali CR to have `spec.deployment.cluster_wide_access` set to `true`). If `false`, all Kiali CRs must set `spec.deployment.cluster_wide_access` to `false`.
- `ANSIBLE_DEBUG_LOGS`: must be `true` or `false`. When `true`, turns on debug logging within the Operator SDK. For details, see the [docs here](https://sdk.operatorframework.io/docs/building-operators/ansible/development-tips/#viewing-the-ansible-logs).
- `ANSIBLE_VERBOSITY_KIALI_KIALI_IO`: Controls how verbose the operator logs are - the higher the value the more output is logged. For details, see the [docs here](https://sdk.operatorframework.io/docs/building-operators/ansible/reference/advanced_options/#ansible-verbosity).
- `ANSIBLE_CONFIG`: must be `/etc/ansible/ansible.cfg` or `/opt/ansible/ansible-profiler.cfg`. If set to `/opt/ansible/ansible-profiler.cfg` a profiler report will be dumped in the operator logs after each reconciliation run.
- `WATCHES_YAML`: must be either (a) `watches-os.yaml`, (b) `watches-os-ns.yaml`, (c) `watches-k8s.yaml` or (d) `watches-k8s-ns.yaml`. If the operator is running on OpenShift, this value must be either (a) or (b); likewise, if the operator is running on a non-OpenShift Kubernetes cluster, this value must be either (c) or (d). If you require the operator to automatically update the Kiali Server with access to new namespaces created in the cluster, set this value to one of the `-ns` files (e.g. `watches-os-ns.yaml` or `watches-k8s-ns.yaml`). This changes the default behavior of the operator such that it will watch for new namespaces getting created and will automatically set up the Kiali Server with the proper access to the new namespace (if such access is to be granted). This namespace watching is not necessary if `spec.deployment.cluster_wide_access` is set to `true` in the Kiali CR.

3. Store the name of the environment variable you want to change in `$ENV_NAME`:

```
ENV_NAME="ANSIBLE_CONFIG"
```
4. Store the new value of the environment variable in `$ENV_VALUE`:

```
ENV_VALUE="/opt/ansible/ansible-profiler.cfg"
```

5. The final step depends on how you installed the Kiali Operator:

{{% alert color="warning" %}}
The commands below assume you are using OpenShift, and as such use `oc`. If you are using a non-OpenShift Kubernetes environment, simply substitute all the `oc` references to `kubectl`.
{{% /alert %}}

- If you installed the operator via helm, simply set the environment variable on the operator deployment directly:

```
oc -n ${OPERATOR_NAMESPACE} set env deploy/kiali-operator "${ENV_NAME}=${ENV_VALUE}"
```

- If you installed the operator via OLM, you must set this environment variable within the operator's CSV and let OLM propagate the new environment variable value down to the operator deployment:

```
oc -n ${OPERATOR_NAMESPACE} patch $(oc -n ${OPERATOR_NAMESPACE} get csv -o name | grep kiali) --type=json -p "[{'op':'replace','path':"/spec/install/spec/deployments/0/spec/template/spec/containers/0/env/$(oc -n ${OPERATOR_NAMESPACE} get $(oc -n ${OPERATOR_NAMESPACE} get csv -o name | grep kiali) -o jsonpath='{.spec.install.spec.deployments[0].spec.template.spec.containers[0].env[*].name}' | tr ' ' '\n' | cat --number | grep ${ENV_NAME} | cut -f 1 | xargs echo -n | cat - <(echo "-1") | bc)/value",'value':"\"${ENV_VALUE}\""}]"
```


### How can I inject an Istio sidecar in the Kiali pod?

By default, Kiali will not have an [Istio sidecar](https://istio.io/latest/docs/setup/additional-setup/sidecar-injection/). If you wish to deploy the Kiali pod with a sidecar, you have to define the `sidecar.istio.io/inject=true` label in the `spec.deployment.pod_labels` setting in the Kiali CR. In addition, to ensure the sidecar and Kiali server containers start in the correct order, the Istio annotation `proxy.istio.io/config` should be defined in the `spec.deployment.pod_annotations` setting in the Kiali CR. For example:

```yaml
spec:
  deployment:
    pod_labels:
      sidecar.istio.io/inject: "true"
    pod_annotations:
      proxy.istio.io/config: '{ "holdApplicationUntilProxyStarts": true }'
```

If you are utilizing CNI in your Istio environment (for example, on OpenShift), Istio will not allow sidecars to work when injected in pods deployed in the control plane namespace, e.g. `istio-system`. [(1)](https://istio.io/v1.10/docs/setup/additional-setup/cni/#identifying-pods-requiring-traffic-redirection) [(2)](https://github.com/istio/istio/issues/34560) [(3)](https://preliminary.istio.io/latest/docs/ops/diagnostic-tools/cni/#diagnose-pod-start-up-failure). In this case, you must deploy Kiali in its own separate namespace. On OpenShift, you can do this using the following instructions.

Determine what namespace you want to install Kiali and create it. Give the proper permissions to Kiali. Create the necessary NetworkAttachmentDefinition. Finally, create the Kiali CR that will tell the operator to install Kiali in this new namespace, making sure to add the proper sidecar injection label as explained earlier.

```
NAMESPACE="kialins"

oc create namespace ${NAMESPACE}

oc adm policy add-scc-to-group privileged system:serviceaccounts:${NAMESPACE}

cat <<EOM | oc apply -f -
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: istio-cni
  namespace: ${NAMESPACE}
EOM

cat <<EOM | oc apply -f -
apiVersion: kiali.io/v1alpha1
kind: Kiali
metadata:
  name: kiali
  namespace: ${NAMESPACE}
spec:
  istio_namespace: istio-system
  auth:
    strategy: anonymous
  deployment:
    pod_labels:
      sidecar.istio.io/inject: "true"
EOM
```

After the operator installs Kiali, confirm you have two containers in your pod. This indicates your Kiali pod has its proxy sidecar successfully injected.

```
$ oc get pods -n ${NAMESPACE}
NAME                    READY   STATUS    RESTARTS   AGE
kiali-56bbfd644-nkhlw   2/2     Running   0          43s
```

### How can I specify a container image digest hash when installing Kiali Server and Kiali Operator?

To tell the operator to install a specific container image using a digest hash, you must use the `deployment.image_digest` setting in conjunction with the `deployment.image_version` setting. `deployment.image_version` is simply the digest hash code and `deployment.image_digest` is the type of digest (most likely you want to set this value to `sha256`). So for example, in your Kiali CR you will want something like this:

```yaml
spec:
  deployment:
    image_version: 63fdb9a9a1aa8fea00015c32cd6dbb63166046ddd087762d0fb53a04611e896d
    image_digest: sha256
```

Leaving `deployment.image_digest` unset or setting it to an empty string will tell the operator to assume the `deployment.image_version` is a tag.

For those that opt not to use the operator to install the server but instead use the server helm chart, the same `deployment.image_version` and `deployment.image_digest` values are supported by the Kiali server helm chart.

As for the operator itself, when installing the operator using its helm chart, the values `image.tag` and `image.digest` are used in the same manner as the `deployment.image_version` and `deployment.image_digest` as explained above. So if you wish to install the operator using a container image digest hash, you will want to use the `image.tag` and `image.digest` in a similar way:

```
helm install --set image.tag=7336eb77199a4d737435a8bf395e1666b7085cc7f0ad8b4cf9456b7649b7d6ad --set image.digest=sha256 ...and the rest of the helm install options...
```

### How can I use a CSI Driver to expose a custom secret to the Kiali Server?
You first must already have a [CSI driver and provider installed](https://secrets-store-csi-driver.sigs.k8s.io/introduction) 
in your cluster and a valid [SecretProviderClass](https://secrets-store-csi-driver.sigs.k8s.io/concepts.html?#secretproviderclass) deployed in the namespace where Kiali is installed. 

To mount a secret exposed by the CSI Driver, you can use the [custom_secret](https://kiali.io/docs/configuration/kialis.kiali.io/#.spec.deployment.custom_secrets) configuration 
to supply the [CSI volume source](https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/volume/#Volume) on the pod. The [Kiali CR reference docs](https://kiali.io/docs/configuration/kialis.kiali.io/#example-cr) have an example. 
The Kiali Operator or server helm chart will automatically expose the secret as a volume mount into the container at the specified mount location.

Although Kiali retrieves the secret over the Kubernetes API, [mounting the secret](https://secrets-store-csi-driver.sigs.k8s.io/topics/sync-as-kubernetes-secret) is required for the CSI Driver to create the backing Kubernetes secret. 
Note that the [`custom_secrets` `optional` flag](https://kiali.io/docs/configuration/kialis.kiali.io/#.spec.deployment.custom_secrets[*].optional) is ignored when mounting secrets from the CSI provider. The secrets are required to exist - then cannot be optional.

### How can I use a secret to pass external service credentials to the Kiali Server?

You can use secrets to store the credentials that Kiali must use to authenticate to external services such as Prometheus. How you configure Kiali is dependent upon whether you install the Kiali Server using the Kiali Operator or the Kiali Server Helm Chart.

**When Using Kiali Operator**

If you are installing using the Kiali Operator, simply set the credential setting to `secret:<secretName>:<secretKey>`. For details, see the [Kiali CR reference docs](https://kiali.io/docs/configuration/kialis.kiali.io/#.spec.external_services).

For example, here is how you can set the bearer token that Kiali will use to authenticate with the Prometheus server.

1. Create a secret with the token.
```
kubectl -n istio-system create secret generic my-secret --from-literal=my-cred=abc123
```
2. Edit the Kiali CR and specify the `token` field with the value `secret:my-secret:my-cred` and specify the type as `bearer` to indicate that authentication will be done with a bearer token.
```yaml
spec:
  external_services:
    prometheus:
      auth:
        type: bearer
        token: secret:my-secret:my-cred
```
At this point, the Kiali Server will soon restart and be reconfigured to authenticate to Prometheus with the given token.

If the secret contains a password, as opposed to a token, set `type` to `basic` to indicate that Kiali should authenticate using `basic` authentication using the given username and password you specify in the configuration:
```yaml
spec:
  external_services:
    prometheus:
      auth:
        type: basic
        username: my-user-name
        password: secret:my-secret:my-cred
```
Note that you can share a secret across multiple external services if they use the same credentials, or you can create multiple secrets if you need to use different credentials for the different external services.

You can use secrets as explained above for the following fields in the Kiali CR:
* `spec.external_services.grafana.auth.username`
* `spec.external_services.grafana.auth.password`
* `spec.external_services.grafana.auth.token`
* `spec.external_services.prometheus.auth.username`
* `spec.external_services.prometheus.auth.password`
* `spec.external_services.prometheus.auth.token`
* `spec.external_services.tracing.auth.username`
* `spec.external_services.tracing.auth.password`
* `spec.external_services.tracing.auth.token`
* `spec.login_token.signing_key`
* `spec.external_services.custom_dashboards.prometheus.auth.username`
* `spec.external_services.custom_dashboards.prometheus.auth.password`
* `spec.external_services.custom_dashboards.prometheus.auth.token`

**When Using Kiali Server Helm Chart**

If you are using the Kiali Server Helm Chart, this feature isn't directly available. However, you can set some configuration options to obtain the same results. Follow the instructions below if you are using the Kiali Server Helm Chart:
1. Create a secret with your password or token in it. Note that the key must be `value.txt`. For example:
```
kubectl -n istio-system create secret generic my-credentials --from-literal=value.txt=abc123xyz789
```

2. Create a Helm values file that (a) defines a custom secret to refer to your secret and mounts it to the place that the Kiali Server expects to see it and (b) tell Kiali to use that secret for the appropriate password or token. For example, if you are setting the Prometheus password, create a `my-values.yaml` file with the following content:

```yaml
deployment:
  custom_secrets:
  - name: "my-credentials"
    mount: "/kiali-override-secrets/prometheus-password"

external_services:
  prometheus:
    auth:
      password: "secret:my-credentials:value.txt"
```

3. Install with the Kiali Server Helm Chart using that values file. For example, to install in the istio-system namespace:
```
helm install -f my-values.yaml -n istio-system kiali-server kiali/kiali-server
```

When you start the Kiali Server, you should now see a debug message in its logs that says:
```
Credentials loaded from secret file [/kiali-override-secrets/prometheus-password/value.txt]
```

NOTE: You must have [enabled logging at the debug level](https://kiali.io/docs/configuration/kialis.kiali.io/#.spec.deployment.logger.log_level) to see the above message in the logs.

This should work with the other credentials that can be read from a mounted secret. They all need to be mounted as a file called `value.txt` that goes into their own sub-directory under `/kiali-override-secrets` - one of:
* grafana-username
* grafana-password
* grafana-token
* prometheus-username
* prometheus-password
* prometheus-token
* tracing-username
* tracing-password
* tracing-token
* login-token-signing-key
* customdashboards-prometheus-username
* customdashboards-prometheus-password
* customdashboards-prometheus-token

So, for example, if you are mounting a custom secret for the Grafana token, the mount location should be declared as `/kiali-override-secrets/grafana-token`.
