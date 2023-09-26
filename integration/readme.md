Install Flux CLI.
```sh
$ curl -s https://fluxcd.io/install.sh | sudo bash
```

Deploy Flux in the running cluster.
```sh
flux install --log-level debug
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

Generate a Flux GitRepository source.
```sh
 flux create source git demo --url=https://github.com/kubeconchina2023/demo --branch=integrate_flux
✚ generating GitRepository source
► applying GitRepository source
✔ GitRepository source created
◎ waiting for GitRepository source reconciliation
✔ GitRepository source reconciliation completed
✔ fetched revision: integrate_flux@sha1:a974a1eedc53604ea28e15a7cdb22ed563bd897e
```

Fetch the generated GitRepository.
```sh
k get gitrepository -n flux-system
NAME   URL                                        AGE    READY   STATUS
demo   https://github.com/kubeconchina2023/demo   7m2s   True    stored artifact for revision 'integrate_flux@sha1:5b258a126058d2fc12430cb186b318ba23d514d1'
```

Generates a Kustomization resource for the demo source created above.
```sh
flux create kustomization demo --source=demo --path=./integration/clusters/staging --interval=20s
✚ generating Kustomization
► applying Kustomization
✔ Kustomization created
◎ waiting for Kustomization reconciliation
✔ Kustomization demo is ready
✔ applied revision integrate_flux@sha1:2a26ef768553795ef74e8d083354ee2f28f214da
```

Verify created kustomizations.
```sh
k get kustomization -n flux-system                         
NAME                 AGE     READY   STATUS
kyverno-controller   6m46s   True    Applied revision: v1.11.0-beta.3@sha256:c78ad83d5e6b9c1cad6e27c548478352895409392474d7f8cb2c8d3253d2ea64
kyverno              6m47s   True    Applied revision: integrate_flux@sha1:5b258a126058d2fc12430cb186b318ba23d514d1
kyverno-policies     6m47s   True    Applied revision: integrate_flux@sha1:5b258a126058d2fc12430cb186b318ba23d514d1
demo                 6m48s   True    Applied revision: integrate_flux@sha1:5b258a126058d2fc12430cb186b318ba23d514d1
```

Verify Kyverno and the policy is installed.
```sh
k get pod -n kyverno    
NAME                                             READY   STATUS    RESTARTS   AGE
kyverno-background-controller-76699f4ccd-ms6ps   1/1     Running   0          7m
kyverno-reports-controller-5886897f45-g8mw8      1/1     Running   0          7m
kyverno-cleanup-controller-5865cd8cd5-xtw7g      1/1     Running   0          7m
kyverno-admission-controller-6b6cc57f5c-kppzx    1/1     Running   0          7m

k get cpol           
NAME            ADMISSION   BACKGROUND   VALIDATE ACTION   READY   AGE     MESSAGE
verify-images   true        true         Enforce           True    6m16s   Ready
```

