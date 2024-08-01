Tanzu Platform for K8s with Openshift 4.13

First, create a new cluster group and copy over all the capabilities from the "run" cluster group.  You can use https://gist.github.com/cdelashmutt-pivotal/fe7a84249689124c5c4dc08dd3fca034 to make the copy of packages easier.

Next, target your new clustergroup and apply the overlay secrets to the Tanzu Platform cluster group:

```
tanzu project use <project>
tanzu operations clustergroup use <cg>

tanzu deploy --only overlays 

KUBECONFIG="$HOME/.config/tanzu/kube/config" kubectl annotate pkgi tcs.tanzu.vmware.com 'ext.packaging.carvel.dev/ytt-paths-from-secret-name.0=tcs-overlay'

KUBECONFIG="$HOME/.config/tanzu/kube/config" kubectl annotate pkgi crossplane.tanzu.vmware.com 'ext.packaging.carvel.dev/ytt-paths-from-secret-name.0=crossplane-overlay'

KUBECONFIG="$HOME/.config/tanzu/kube/config" kubectl annotate pkgi egress.tanzu.vmware.com 'ext.packaging.carvel.dev/ytt-paths-from-secret-name.0=egress-overlay'

KUBECONFIG="$HOME/.config/tanzu/kube/config" kubectl annotate pkgi config-server.spring.tanzu.vmware.com 'ext.packaging.carvel.dev/ytt-paths-from-secret-name.0=config-server-overlay'

KUBECONFIG="$HOME/.config/tanzu/kube/config" kubectl annotate pkgi spring-cloud-gateway.tanzu.vmware.com 'ext.packaging.carvel.dev/ytt-paths-from-secret-name.0=scg-overlay'
```

We will need a custom trait to install this in each space NS in the target cluster once we can add custom traits, but you can manually apply them to each NS for now.
```
---
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: istio-cni
  namespace: <target-namespace>
  labels:
    operator.istio.io/component: "Cni"
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:openshift:scc:restricted-v2:<target-namespace>
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:openshift:scc:restricted-v2
subjects:
- kind: Group
  name: system:serviceaccounts:<target-namespace>
```

Then you should be able to deploy apps to your space running on Openshift.