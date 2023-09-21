```sh
$ curl -s https://fluxcd.io/install.sh | sudo bash
```


```sh
$ flux install --log-level debug
✚ generating manifests
✔ manifests build completed
► installing components in flux-system namespace
CustomResourceDefinition/alerts.notification.toolkit.fluxcd.io created
CustomResourceDefinition/buckets.source.toolkit.fluxcd.io created
CustomResourceDefinition/gitrepositories.source.toolkit.fluxcd.io created
CustomResourceDefinition/helmcharts.source.toolkit.fluxcd.io created
CustomResourceDefinition/helmreleases.helm.toolkit.fluxcd.io created
CustomResourceDefinition/helmrepositories.source.toolkit.fluxcd.io created
CustomResourceDefinition/kustomizations.kustomize.toolkit.fluxcd.io created
CustomResourceDefinition/ocirepositories.source.toolkit.fluxcd.io created
CustomResourceDefinition/providers.notification.toolkit.fluxcd.io created
CustomResourceDefinition/receivers.notification.toolkit.fluxcd.io created
Namespace/flux-system created
ResourceQuota/flux-system/critical-pods-flux-system created
ServiceAccount/flux-system/helm-controller created
ServiceAccount/flux-system/kustomize-controller created
ServiceAccount/flux-system/notification-controller created
ServiceAccount/flux-system/source-controller created
ClusterRole/crd-controller-flux-system created
ClusterRole/flux-edit-flux-system created
ClusterRole/flux-view-flux-system created
ClusterRoleBinding/cluster-reconciler-flux-system created
ClusterRoleBinding/crd-controller-flux-system created
Service/flux-system/notification-controller created
Service/flux-system/source-controller created
Service/flux-system/webhook-receiver created
Deployment/flux-system/helm-controller created
Deployment/flux-system/kustomize-controller created
Deployment/flux-system/notification-controller created
Deployment/flux-system/source-controller created
NetworkPolicy/flux-system/allow-egress created
NetworkPolicy/flux-system/allow-scraping created
NetworkPolicy/flux-system/allow-webhooks created
◎ verifying installation
✔ helm-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ notification-controller: deployment ready
✔ source-controller: deployment ready
✔ install finished
```

```sh
$ flux create source git flux-system \
> --url=https://github.com/fluxcd/flux2-multi-tenancy \
> --branch=update-kyverno \
✚ generating GitRepository source
► applying GitRepository source
✔ GitRepository source created
◎ waiting for GitRepository source reconciliation
✔ GitRepository source reconciliation completed
✔ fetched revision: update-kyverno@sha1:1471ec21ec0ed77893841cab6a19c2fe65f5ae4b
```

```sh
$ flux create kustomization flux-system \
>   --source=flux-system \
>   --path=./clusters/staging
✚ generating Kustomization
► applying Kustomization
✔ Kustomization created
◎ waiting for Kustomization reconciliation
✔ Kustomization flux-system is ready
✔ applied revision update-kyverno@sha1:1471ec21ec0ed77893841cab6a19c2fe65f5ae4b
```

```sh
$ kubectl -n flux-system wait kustomization/kyverno --for=condition=ready --timeout=5m
kustomization.kustomize.toolkit.fluxcd.io/kyverno condition met

$ kubectl -n flux-system wait kustomization/kyverno-controller --for=condition=ready --timeout=3m
kustomization.kustomize.toolkit.fluxcd.io/kyverno-controller condition met

$ kubectl -n flux-system wait kustomization/kyverno-policies --for=condition=ready --timeout=3m
kustomization.kustomize.toolkit.fluxcd.io/kyverno-policies condition met

$ kubectl -n flux-system wait kustomization/tenants --for=condition=ready --timeout=3m
kustomization.kustomize.toolkit.fluxcd.io/tenants condition met
```

```sh
$ flux get all --all-namespaces
NAMESPACE  	NAME                            	REVISION               	SUSPENDED	READY	MESSAGE
flux-system	ocirepository/kyverno-controller	v1.10.3@sha256:01a5a07c	False    	True 	stored artifact for digest 'v1.10.3@sha256:01a5a07c'

NAMESPACE  	NAME                     	REVISION                    	SUSPENDED	READY	MESSAGE
apps       	gitrepository/dev-team   	dev-team@sha1:0b99b5c4      	False    	True 	stored artifact for revision 'dev-team@sha1:0b99b5c4'
flux-system	gitrepository/flux-system	update-kyverno@sha1:1471ec21	False    	True 	stored artifact for revision 'update-kyverno@sha1:1471ec21'

NAMESPACE	NAME                  	REVISION       	SUSPENDED	READY	MESSAGE
apps     	helmrepository/podinfo	sha256:b6bb2802	False    	True 	stored artifact: revision 'sha256:b6bb2802'

NAMESPACE	NAME                  	REVISION	SUSPENDED	READY	MESSAGE
apps     	helmchart/apps-podinfo	6.4.1   	False    	True 	pulled 'podinfo' chart with version '6.4.1'

NAMESPACE	NAME               	REVISION	SUSPENDED	READY	MESSAGE
apps     	helmrelease/podinfo	6.4.1   	False    	True 	Release reconciliation succeeded

NAMESPACE  	NAME                            	REVISION                    	SUSPENDED	READY	MESSAGE
apps       	kustomization/dev-team          	dev-team@sha1:0b99b5c4      	False    	True 	Applied revision: dev-team@sha1:0b99b5c4
flux-system	kustomization/flux-system       	update-kyverno@sha1:1471ec21	False    	True 	Applied revision: update-kyverno@sha1:1471ec21
flux-system	kustomization/kyverno           	update-kyverno@sha1:1471ec21	False    	True 	Applied revision: update-kyverno@sha1:1471ec21
flux-system	kustomization/kyverno-controller	v1.10.3@sha256:01a5a07c     	False    	True 	Applied revision: v1.10.3@sha256:01a5a07c
flux-system	kustomization/kyverno-policies  	update-kyverno@sha1:1471ec21	False    	True 	Applied revision: update-kyverno@sha1:1471ec21
flux-system	kustomization/tenants           	update-kyverno@sha1:1471ec21	False    	True 	Applied revision: update-kyverno@sha1:1471ec21
```